# Model
```
                  ┌─────────────────┐
                  │    workflow     │
                  │─────────────────│
                  │ id              │
                  │ owner_user_id   │
                  └───────┬─────────┘
                          │ 1
               ┌──────────┴───────────┐
               │                      │
               │ n                    │ n
               ▼                      ▼
    ┌───────────────────┐   ┌──────────────────────┐
    │ workflow_member   │   │ workflow_invitation  │
    │───────────────────│   │──────────────────────│
    │ workflow_id       │   │ workflow_id          │
    │ user_id           │   │ invited_user_id      │
    │ role              │   │ invited_by           │
    │ created_by        │   │ role                 │
    └───────────────────┘   │ status               │
                            └──────────────────────┘
```

```Java
public enum WorkflowRole {
    EDITOR,
    VIEWER
}

public enum InvitationStatus {
    PENDING,
    ACCEPTED,
    DECLINED,
    REVOKED,
    EXPIRED
}

public enum WorkflowPermission {
    VIEW,
    EDIT,
    EXECUTE,
    MANAGE_MEMBERS,
    DELETE
}
```

# Records
```Java
public record WorkflowMemberDto(
    Long id,
    String userId,
    WorkflowRole role,
    Instant createdAt,
    String createdBy
) {}

public record WorkflowInvitationDto(
    Long id,
    Long workflowId,
    String invitedUserId,
    String invitedBy,
    WorkflowRole role,
    InvitationStatus status,
    Instant createdAt,
    Instant expiresAt,
    Instant acceptedAt
) {}

public record CreateWorkflowInvitationRequest(
    String userId,
    WorkflowRole role
) {}

public record UpdateWorkflowMemberRoleRequest(
    WorkflowRole role
) {}
```

## Response
```Json
{
  "id": 123,
  "name": "Rapportage workflow",
  "description": "Genereert kwartaalrapportage",
  "ownerUserId": "1f3d...",
  "access": {
    "owner": false,
    "role": "EDITOR",
    "permissions": [
      "VIEW",
      "EDIT",
      "EXECUTE"
    ]
  }
}
```


```sql
workflow
--------
id                  PK
name
description
owner_user_id       Keycloak sub
created_at
updated_at
...

workflow_member
---------------
id                  PK
workflow_id         FK -> workflow.id
user_id             Keycloak sub
role                EDITOR / eventueel VIEWER
created_at
created_by

UNIQUE(workflow_id, user_id)

workflow_invitation
-------------------
id                  PK
workflow_id         FK -> workflow.id
invited_user_id     Keycloak sub
invited_by          Keycloak sub
role                EDITOR / eventueel VIEWER
status              PENDING / ACCEPTED / DECLINED / REVOKED / EXPIRED
created_at
expires_at
accepted_at
```

### workflow_member
```sql
-- Vxx__create_workflow_member.sql

CREATE TABLE workflow_member (
    id BIGSERIAL PRIMARY KEY,

    workflow_id BIGINT NOT NULL,
    user_id VARCHAR(255) NOT NULL,

    role VARCHAR(32) NOT NULL,

    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    created_by VARCHAR(255) NOT NULL,

    CONSTRAINT fk_workflow_member_workflow
        FOREIGN KEY (workflow_id)
        REFERENCES workflow(id)
        ON DELETE CASCADE,

    CONSTRAINT uq_workflow_member_workflow_user
        UNIQUE (workflow_id, user_id)
);

CREATE INDEX idx_workflow_member_user_id
    ON workflow_member(user_id);

CREATE INDEX idx_workflow_member_workflow_id
    ON workflow_member(workflow_id);
```
### inventations
```sql
CREATE TABLE workflow_invitation (
    id BIGSERIAL PRIMARY KEY,

    workflow_id BIGINT NOT NULL,

    invited_user_id VARCHAR(255) NOT NULL,
    invited_by VARCHAR(255) NOT NULL,

    role VARCHAR(32) NOT NULL,
    status VARCHAR(32) NOT NULL,

    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    accepted_at TIMESTAMP WITH TIME ZONE,

    CONSTRAINT fk_workflow_invitation_workflow
        FOREIGN KEY (workflow_id)
        REFERENCES workflow(id)
        ON DELETE CASCADE,

    CONSTRAINT chk_workflow_invitation_role
        CHECK (role IN ('EDITOR', 'VIEWER')),

    CONSTRAINT chk_workflow_invitation_status
        CHECK (
            status IN (
                'PENDING',
                'ACCEPTED',
                'DECLINED',
                'REVOKED',
                'EXPIRED'
            )
        )
);

CREATE INDEX idx_workflow_invitation_invited_user
    ON workflow_invitation(invited_user_id);

CREATE INDEX idx_workflow_invitation_workflow
    ON workflow_invitation(workflow_id);

CREATE UNIQUE INDEX uq_workflow_invitation_pending
    ON workflow_invitation(workflow_id, invited_user_id)
    WHERE status = 'PENDING';
```

```Java
public enum InvitationStatus {
    PENDING,
    ACCEPTED,
    DECLINED,
    REVOKED,
    EXPIRED
}
```

```Java
public boolean canEdit(Long workflowId, CurrentUser user) {

    Workflow workflow = workflowRepository
        .findById(workflowId)
        .orElseThrow(...);

    if (isApplicationAdmin(user)) {
        return true;
    }

    if (workflow.isOwner(user.id())) {
        return true;
    }

    return memberRepository.existsByWorkflowIdAndUserIdAndRoleIn(
        workflowId,
        user.id(),
        Set.of(WorkflowRole.EDITOR)
    );
}
```

```Java
public void requireEdit(Long workflowId, CurrentUser user) {
    if (!canEdit(workflowId, user)) {
        throw new AccessDeniedException(
            "User is not allowed to edit workflow"
        );
    }
}
```

Registreer wie workflow heeft uitgevoerd:
```Java
public record WorkflowExecutionContext(
    Long workflowId,
    String initiatedBy,
    String workflowOwner
) {}
```

# API ontwerp
* GET /api/workflows/{workflowId}/members
* POST /api/workflows/{workflowId}/invitations
* GET /api/workflow-invitations
* POST /api/workflow-invitations/{id}/accept
* POST /api/workflow-invitations/{id}/decline
* DELETE /api/workflows/{workflowId}/invitations/{id}
* DELETE /api/workflows/{workflowId}/members/{userId}
* PATCH /api/workflows/{workflowId}/members/{userId}

## Java model
```
                         ┌─────────────────────┐
                         │      Keycloak       │
                         │                     │
                         │ Authentication      │
                         │ Global roles        │
                         └──────────┬──────────┘
                                    │ JWT
                                    ▼
                         ┌─────────────────────┐
                         │   Spring Security   │
                         └──────────┬──────────┘
                                    │
                              CurrentUser
                                    │
              ┌─────────────────────┴──────────────────────┐
              │                                            │
              ▼                                            ▼
 ┌──────────────────────────┐                 ┌───────────────────────┐
 │ WorkflowAuthorizationSvc │                 │ WorkflowSharingSvc    │
 │                          │                 │                       │
 │ VIEW                     │                 │ invite                │
 │ EDIT                     │                 │ accept                │
 │ EXECUTE                  │                 │ revoke                │
 │ MANAGE_MEMBERS           │                 │ remove member         │
 │ DELETE                   │                 │ change role           │
 └─────────────┬────────────┘                 └───────────┬───────────┘
               │                                          │
               └──────────────────┬───────────────────────┘
                                  ▼
                    ┌────────────────────────┐
                    │       PostgreSQL       │
                    │                        │
                    │ workflow               │
                    │ workflow_member        │
                    │ workflow_invitation    │
                    └────────────┬───────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │ Workflow execution     │
                    │                        │
                    │ owner                  │
                    │ initiatedBy            │
                    │ execution identity     │
                    └────────────┬───────────┘
                                 │
                                 ▼
                       Impersonation Service
```

```
Keycloak
    authentication
    ROLE_USER / ROLE_ADMIN

Application DB
    workflow.owner_user_id

    workflow_member
        workflow_id
        user_id
        role = EDITOR

    workflow_invitation
        workflow_id
        user_id
        role
        status

WorkflowAuthorizationService
    requireView()
    requireEdit()
    requireExecute()
    requireManageMembers()
    requireDelete()

WorkflowSharingService
    invite()
    accept()
    decline()
    revoke()
    removeMember()
```

> Een gedeelde workflow uitvoeren mag niet automatisch betekenen dat de editor de security-identiteit van de workflow-owner gebruikt.

