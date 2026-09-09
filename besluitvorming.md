> [!TIP]
> * Begrijpen, Wegen, Besluiten

> [!INFO]
> * productie herstellen
> * impact communiceren
> * feiten verzamelen
> * geen schuldige aanwijzen
> * wijzigingsproces onderzoeken
> * medior ondersteunen
> * tijdelijke risicomaatregelen
> * peer review verbeteren
> * duidelijke verantwoordelijkheden
> * blameless postmortem
> * leren en opvolgen

> [!IMPORTANT]
> - Flow 
>    - Doel scherpstellen (welk probleem lossen we op)
>    - Bepaal impact en urgentie (onderscheidt belangrijk en urgent. Wat gebeurt er als we niets doen)
>    - Informatie (verklein onzekerheid door informatie te verzamelen. Te weinig is impulsief, te veel is besluiteloos)
>    - Mensen (betrek de juiste mensen met kennis. Je hoeft niet zelf alles te doen of te kennen)
>    - Alternatieven en trade-offs (Opties expliciet maken. Optie A is dit, Optie B dit maar complexer etc)
>    - Risico en omkeerbaarheid (bepaal tempo door de impact met omkeerbaarheid te combineren)
>    - Eigenaarschap (We kiezen X vanwege A en B; Risico C accepteren of mitigeren we op deze manier)
>    - Proactiviteit (Vooruitkijken; vraag wat kan misgaan, welke afhankelijkheden zie ik, wie moet dit weten, welke monitoring is nodig)
>    - Uitvoeren en bijsturen (besluit is geen eindpunt; besluiten, handelen, resultaat, observeren, leren, bijsturen)

>[!CAUTION]
> Assessment waarschijnlijk in de volgende opzet
> - Informatie verwerken -> structureren -> prioriteren -> afwegen -> besluiten -> onderbouwen -> communiceren

## Snelle weergave
> Waarom? → Wanneer? → Kan het? → Is het verstandig? → Met wie? → Welke keuzes? → Eerst toetsen? → Besluiten → Beheerst uitvoeren → Leren

# De kapstok
| Fase | Aspect | Centrale vraag |
|---|---|---|
| **1. Waarom?** | **Doel** | Welk probleem moeten we daadwerkelijk oplossen? Wat is de gewenste uitkomst? |
| | **Businesswaarde** | Welke waarde levert dit op? Wat zijn de gevolgen als we niets veranderen? |
| **2. Wanneer?** | **Tijd / urgentie** | Wanneer moet er iets gebeuren? Wat gebeurt er als we wachten? Moeten we nú handelen of hebben we tijd voor verdere analyse? |
| **3. Kan het?** | **Technische haalbaarheid** | Is het technisch realiseerbaar binnen de bestaande omgeving, kennis en randvoorwaarden? |
| | **Security** | Welke security-, compliance- en privacyrisico's ontstaan en hoe kunnen we die mitigeren? |
| | **Blast radius** | Wat is de maximale impact als de oplossing, wijziging of component faalt? |
| | **Tenant isolation** | Zijn gebruikers, teams, workloads en data voldoende van elkaar geïsoleerd? |
| | **Beschikbaarheid** | Welke beschikbaarheid is vereist en hoe realiseren we redundantie en herstel? |
| **4. Is het verstandig?** | **Kosten** | Wat zijn implementatie-, infrastructuur- en structurele beheerkosten? |
| | **Beheerbaarheid** | Kunnen we dit betrouwbaar monitoren, patchen, upgraden, troubleshooten en ondersteunen? |
| | **Migratierisico** | Welke risico's ontstaan tijdens de overgang en hoe beperken we die? |
| **5. Met wie?** | **Stakeholders** | Wie wordt geraakt? Wie bezit relevante expertise? Wie heeft mandaat om te beslissen? |
| **6. Wat zijn de keuzes?** | **Alternatieven** | Welke alternatieven bestaan er, inclusief niets doen? Wat zijn de belangrijkste trade-offs? |
| **7. Kunnen we onzekerheid verkleinen?** | **Pilot / PoC** | Kunnen we belangrijke aannames eerst kleinschalig, meetbaar en omkeerbaar toetsen? |
| **8. Beslis** | **Besluitmoment** | Hebben we voldoende informatie? Wat besluiten we, waarom en wie neemt verantwoordelijkheid? |
| **9. Voer beheerst uit** | **Rollback** | Wat doen we wanneer de implementatie niet werkt? Wanneer stoppen we en wanneer draaien we terug? |
| **10. Leer** | **Monitoren / bijsturen** | Werkt de beslissing zoals verwacht? Welke signalen volgen we en wanneer heroverwegen we de keuze? |

## tijd/urgentie
```
                    URGENTIE
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
         HOOG                      LAAG
          │                         │
    sneller handelen         meer tijd voor
          │                  analyse/afstemming
          ↓                         ↓
     OMKEERBAAR?               GROTE IMPACT?
       /      \                  /       \
     JA       NEE               JA       NEE
      ↓         ↓                ↓         ↓
 tijdelijk   risico           zorgvuldig  relatief
 ingrijpen   beperken         valideren   eenvoudig
```
> Besluitvaardigheid is niet zo snel mogelijk beslissen, maar op het juiste moment voldoende onderbouwd besluiten én vervolgens handelen.
