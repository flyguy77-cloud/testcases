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
    OWNER,
    EDITOR,
    VIEWER
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

```sql
CREATE TABLE workflow_member (
    id BIGSERIAL PRIMARY KEY,

    workflow_id BIGINT NOT NULL,
    user_id VARCHAR(255) NOT NULL,

    role VARCHAR(32) NOT NULL,

    created_at TIMESTAMP NOT NULL,
    created_by VARCHAR(255) NOT NULL,

    CONSTRAINT fk_workflow_member_workflow
        FOREIGN KEY (workflow_id)
        REFERENCES workflow(id)
        ON DELETE CASCADE,

    CONSTRAINT uq_workflow_member
        UNIQUE (workflow_id, user_id)
);
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

POST /api/workflows/{workflowId}/invitations

Request
```Json
{
  "userId": "keycloak-sub-123",
  "role": "EDITOR"
}
```

POST /api/workflow-invitations/{invitationId}/accept

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

API ontwerp
* GET /api/workflows/{workflowId}/members
* POST /api/workflows/{workflowId}/invitations
* GET /api/workflow-invitations
* POST /api/workflow-invitations/{id}/accept
* POST /api/workflow-invitations/{id}/decline
* DELETE /api/workflows/{workflowId}/invitations/{id}
* DELETE /api/workflows/{workflowId}/members/{userId}
* PATCH /api/workflows/{workflowId}/members/{userId}

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

