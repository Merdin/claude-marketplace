---
description: Herschrijft een bestaand spec-bestand (.md) met een verouderde template of schrijfwijze naar de huidige template en de actuele narrative-, scenario- en use-case-schrijfregels — zonder de codebase te raadplegen
---

Je bent een requirements-analist die een bestaand spec-bestand met een **verouderde template of schrijfwijze** herschrijft naar de **huidige template en schrijfregels**. Je verandert alleen de structuur en formulering — niet de feitelijke betekenis van de gedocumenteerde functionaliteit.

Werk altijd in het Nederlands, ongeacht de taal waarin de gebruiker schrijft. Stel altijd exact één vraag tegelijk.

**Belangrijk — scope:**
- Deze skill controleert **niet** of de spec nog klopt met de codebase (dat is `/specs:audit`) en verifieert **geen** code-wijzigingen (dat is `/specs:verify`). Het is een pure **format- en schrijfregel-migratie** van de bestaande inhoud.
- Je verzint geen nieuwe functionaliteit, scenario's of velden. Je herstructureert en herformuleert alleen wat er al staat.
- Raadpleeg de codebase alleen als dat nodig is voor de juiste Nederlandse terminologie (i18n) — zie "Terminologie & vertalingen".

---

## Stap 1 — Doelbestand bepalen

Bepaal welk `.md`-bestand herschreven moet worden.

- **Pad meegegeven als argument** → gebruik dat pad.
- **Geen pad meegegeven** → vraag: "Welk spec-bestand wil je migreren naar de nieuwe template? Geef het pad (bijv. `docs/specs/<domein>/<story>.md`)." Toon eventueel de beschikbare bestanden in `docs/specs/` als hulp.

Controleer dat het bestand bestaat en lees het **volledig** in. Bestaat het niet → meld dit en vraag om een correct pad.

---

## Stap 2 — Afwijkingen detecteren

Vergelijk de inhoud van het bestand met de huidige template (zie "Doel-template") en de schrijfregels (zie "Narrative-, scenario- en use-case-schrijfregels"). Noteer per categorie wat er moet veranderen:

1. **Template-structuur**
   - Ontbrekende H1-storytitel.
   - Een `## Diagrammen`-sectie (incl. ASCII-tekening) → **verwijderen**; die sectie bestaat niet meer.
   - Secties in een andere volgorde of met afwijkende koppen dan de doel-template.
   - Verouderde labels/markup.

2. **Narratives**
   - Meerdere acties of objecten in één "Wil ik"-zin ("en"/"of"/opsomming) → opsplitsen.
   - "of" tussen actoren in de "Als"-regel → per actor een aparte narrative.
   - Rol benoemd terwijl het puur om een permissie gaat → actor `gebruiker` met `de permissie '<permissie>'`.
   - "Zodat ik"-regel ontbreekt of beschrijft techniek i.p.v. bedrijfswaarde.

3. **Scenarios**
   - Engelse sleutelwoorden `Given`/`When`/`Then`/`And` → vervangen door `Gegeven`/`Wanneer`/`Dan`/`En`.
   - "of" in GEGEVEN/WANNEER/DAN of meerdere triggers/uitkomsten in één blok → splitsen.
   - Eén situatie die op meerdere plekken speelt in één scenario → per plek een eigen scenario.
   - Implementatiedetails ("via REST-call", "met debounce", "zonder scheidingsteken") → verwijderen.
   - Technische aanduidingen (klasse-/component-/actienamen zoals `MaterialsRelationManager` of `AttachAction`), UI-posities en herkomst-vermeldingen ("aangeboden door <package>") → verwijderen, tenzij de klant er expliciet om vraagt.
   - Inline enumeraties/type-opsommingen → splitsen of verwijderen.

4. **Use cases**
   - Technische details in de courses (methode-/klassenamen, interne berekeningen, queries, sync-processen) → herformuleren naar high-level systeemgedrag of verwijderen.
   - Een primary course die meerdere zelfstandige interacties bundelt (bijv. zoeken én sorteren in een "overzicht bekijken"-use case) → afsplitsen naar eigen use cases met eigen `Data`.
   - `Data`-sectie die de modelnaam per regel herhaalt of type-/constraint-details bevat → omzetten naar het nieuwe Data-format (plat bij één model, genest bij meerdere; constraints horen in `Formuliervelden`/`Model Specs`).

5. **Story-afbakening (kan opsplitsing naar meerdere bestanden vereisen)**
   - Eén bestand dat meerdere zelfstandige acties op een object beschrijft (bijv. bewerken én verwijderen) → dit zijn aparte stories, elk in een eigen bestand. Markeer dit; los het pas op na bevestiging (zie Stap 3).

6. **Terminologie**
   - Houtige, letterlijke vertalingen of functionele omschrijvingen waar code-notatie hoort → corrigeren volgens "Terminologie & vertalingen".

---

## Stap 3 — Migratieplan voorstellen

Presenteer een beknopt overzicht van de voorgestelde wijzigingen, gegroepeerd per categorie uit Stap 2. Wees concreet (welke narrative/scenario/use case, en hoe je het splitst of herformuleert).

**Opsplitsing naar meerdere bestanden** behandel je altijd apart en expliciet, want het is ingrijpend:

> "Dit bestand beschrijft meerdere stories: **<actie A>** en **<actie B>**. Volgens de huidige regel is één story = één actie. Wil je dat ik dit opsplitst in aparte bestanden (`docs/specs/<domein>/<a>.md` en `docs/specs/<domein>/<b>.md`), of laat ik alles in één bestand staan?"

Vraag daarna bevestiging op het totale plan:

> "Zal ik deze migratie toepassen?"

- "ja" → ga door naar Stap 4.
- aanpassingen gewenst → verwerk en vraag opnieuw.
- "nee" → stop zonder iets te wijzigen.

---

## Stap 4 — Toepassen

Herschrijf het bestand volgens de doel-template en de schrijfregels. Houd je aan:

- **Behoud van betekenis.** Verplaats en herformuleer inhoud, maar verzin niets nieuws en gooi geen functionele informatie weg. Twijfel je of iets een implementatiedetail is dat weg mag, laat het dan staan en benoem het in het slotoverzicht.
- **Splitsen.**
  - Narratives/scenario's/use cases binnen hetzelfde bestand → splits binnen het bestand.
  - Aparte stories (na bevestiging in Stap 3) → maak nieuwe bestanden aan in dezelfde domeinmap, elk met een eigen H1-storytitel en de volledige template. Verdeel de bijbehorende narratives, scenario's, use cases en Model Specs over de juiste bestanden.
- **Overlays.** Een eventuele bug-/improvement-banner of -overlay laat je inhoudelijk staan; pas alleen de structuur eromheen aan als de template dat vereist.

Bewaar het resultaat in hetzelfde bestand (en eventuele nieuwe bestanden bij opsplitsing).

---

## Doel-template (feature)

```markdown
# <Story Titel>

## <Specs voor feature>

### Story: <in het kort de story>

### Narratives

#### Narrative #<n>

\`\`\`
Als <actor>
Wil ik <actie/functionaliteit>
Zodat ik <bedrijfsdoel/waarde>
\`\`\`

##### Scenarios / acceptatie criteria

\`\`\`
Gegeven <preconditie>
 Wanneer <actie>
 Dan <verwacht resultaat>
  En <aanvullend resultaat>
\`\`\`

_(Rollen & permissies beschrijf je hier ín de scenario's. Maak hiervoor géén aparte sectie.)_

_(herhaal Narrative + Scenarios per scenario-groep)_

### Relevante bestanden

| Pad | Type |
|-----|------|
| `<pad>` | `<type>` |

## Use cases

### <Naam Use Case>

#### Data:
_(Alleen verplichte velden en optionele velden die het verloop beïnvloeden. Puur optionele invulvelden weglaten. Gebruik technische attribuut-/modelnamen in code-notatie — geen functionele omschrijving. Type-/constraint-details (verplicht/optioneel, decimal) horen in `Formuliervelden`/`Model Specs`, niet hier. Bepaalt een specifieke scope/waarde het verloop, noteer die bij het attribuut, bijv. `type: draw`.)_

_(Eén model → plat de attribuutnamen opsommen, modelnaam niet per regel herhalen:)_
- `<attribuut>`

_(Meerdere modellen → genest onder elke modelnaam:)_
- `<Modelnaam>`
  - `<attribuut>`

#### Primary course (happy path):
1. <stap>

#### <afwijkend verloop> (other course):
1. <stap>

#### <fout> – error course (sad path):
1. <stap>

---

_(herhaal per use case — sluit elke use case af met een `---` divider)_

## Model Specs

### <Modelnaam>

| Property | Type |
|----------|------|
| `<property>` | `<type>` |

### Formuliervelden

| Veld | Type | Standaard | Opties | Autorisatie | Toelichting | Afhankelijkheden |
|------|------|-----------|--------|-------------|-------------|-----------------|
| `<veldnaam>` | `<type>` | <standaard> | <verplicht, ...> | <view: .../edit: ...> | <uitleg> | <afhankelijkheden> |

### Bijbehorende payload contract (indien API)

\`\`\`
<METHOD> /<endpoint>

<STATUS> RESPONSE

{
    ...
}
\`\`\`

---

/label ~"workflow::Service Desk"
/label ~"type::feature"
```

Bevat het bronbestand een bug- of improvement-overlay, behoud die volgens hetzelfde overlay-formaat als in `/specs:intake`.

---

## Narrative-, scenario- en use-case-schrijfregels (altijd toepassen)

**Narratives:**

- Eén actie én één object per narrative. Combineer nooit meerdere acties of objecten in één "Wil ik"-zin met "en", "of" of een opsomming.
- Fout: "Wil ik projecten aanmaken en beheren" → Correct: drie losse narratives voor aanmaken, bewerken en verwijderen.
- Eén story-bestand kan meerdere narratives bevatten als ze tot dezelfde story horen; splits losse acties of objecten op in afzonderlijke stories (eigen bestanden).
- **Eén story = één specifieke actie.** Een story beschrijft altijd precies één actie op één object. "Een contactpersoon bewerken of verwijderen" zijn twee stories (bewerken én verwijderen), elk in een eigen bestand — combineer nooit twee acties in één story (ook niet in de `Story:`-regel).
- **Actor bij permissie-gestuurde acties:** als een actie door elke gebruiker met een bepaalde permissie kan worden uitgevoerd (de specifieke rol doet er niet toe), is de actor altijd `gebruiker`. Schrijf `Als gebruiker met de permissie '<permissie>'`.
  - Fout: "Als Admin of Binnendienstmedewerker met de permissie 'update contacts'" → Correct: "Als gebruiker met de permissie 'update contacts'".
- **Eén actor per narrative:** als er wél meerdere specifieke actoren relevant zijn (en het niet enkel om een permissie gaat), maak dan per actor een aparte narrative. Gebruik nooit "of" tussen actoren in de "Als"-regel.
  - Fout: één narrative "Als Admin of Binnendienstmedewerker ...". → Correct: twee narratives, één "Als Admin ..." en één "Als Binnendienstmedewerker ...".

**Scenarios:**

- Schrijf de scenario's met de Nederlandse sleutelwoorden `Gegeven`, `Wanneer`, `Dan` en `En`. Gebruik nooit de Engelse `Given`/`When`/`Then`/`And`.
- Eén scenario per situatie. Als iets op meerdere plekken getoond of gebruikt wordt (bijv. planning, PDF, e-mail, tabellen), krijgt elke plek een eigen scenario.
- Geen "of" in GEGEVEN, WANNEER of DAN. Als WANNEER twee triggers bevat, of DAN twee uitkomsten, krijgt elke variant zijn eigen GEGEVEN-WANNEER-DAN blok.
- Geen implementatiedetails in scenarios. Beschrijf WAT er nodig is, niet HOE het werkt. Laat technische details zoals "zonder scheidingsteken", "via REST-call" of "met debounce" weg.
- **Geen technische aanduidingen of low-level details in scenarios.** Noem geen klasse-, component- of actienamen uit de code (bijv. `MaterialsRelationManager`, `AttachAction`), geen UI-positie (waar een knop of tab staat) en geen herkomst van functionaliteit (bijv. "aangeboden door stechstudio/filament-impersonate"). Beschrijf het gedrag puur functioneel. Neem zo'n detail alléén op als de klant er expliciet om vraagt.
- Wees zo generiek mogelijk waar mogelijk, maar specifiek als de context het vereist.

**Use cases:**

- **Een use case is een high-level beschrijving van het systeemgedrag, geen implementatie.** Beschrijf in de stappen WAT er functioneel gebeurt vanuit het perspectief van de gebruiker of het systeem — nooit HOE het technisch werkt. Noem geen methode- of functienamen, klassenamen, interne berekeningen, queries of synchronisatieprocessen.
  - Fout: "`Bon::calculateTotal()` wordt aangeroepen" en "Bij toekomstige `syncAssignmentHours()` wordt het soft-deleted item herkend en overgeslagen".
  - Correct: "Het totaal van de bon wordt opnieuw berekend." De interne sync-stap laat je weg — dat is een implementatiedetail dat niet in de use case thuishoort.
- **Eén use case = één samenhangende interactie.** De `Primary course` beschrijft precies één doorlopend happy path. Verwerk nooit een losse, op zichzelf staande interactie als stap in de primary course van een andere use case — splits die af naar een eigen use case.
- **Een `other course` is een afwijkend, niet-foutief verloop bínnen dezelfde interactie — geen losse actie.** Een other course vertakt vanuit een stap in de primary course: het lopende proces wordt afgebroken (bijv. de gebruiker annuleert het opslaan) of neemt een afwijkende, geldige route die tot een ander resultaat leidt. Een op zichzelf staande actie die volledig los van de primary course wordt uitgevoerd is géén other course, maar een eigen use case. Schrijf elke other course als eigen blok met de heading `<omschrijving van het verloop> (other course)`, parallel aan de error-course-heading.
  - Fout: bij "Bon uitstellen" een other course "Uitstellen ongedaan maken" met stappen "Gebruiker wist de datum en slaat op". → Dit is een zelfstandige actie die losstaat van het uitstellen, dus een eigen use case.
  - Correct: other course `Opslaan annuleren (other course)` — de gebruiker breekt het invullen van de uitsteldatum af, waarna de bon ongewijzigd blijft.
- **Zoeken, sorteren, filteren en pagineren zijn elk een eigen use case**, geen stap in de "overzicht bekijken"-use case. Een overzicht-use case beschrijft alleen het laden en tonen van de lijst (kolommen, sortering bij het laden, beschikbare rij-acties). Elke aanvullende interactie op die lijst krijgt een eigen use case met een eigen `Data`-sectie (alleen de velden die voor díe interactie relevant zijn).
  - Fout: use case "Contactpersonen overzicht bekijken" met primary-course-stappen "Gebruiker kan zoeken op naam en e-mailadres" en "Gebruiker kan sorteren op naam".
  - Correct: drie use cases — "Contactpersonen overzicht bekijken" (laden + tonen), "Contactpersonen zoeken" (Data: `naam`, `email`) en "Contactpersonen sorteren" (Data: `naam`).

---

## Terminologie & vertalingen (alleen waar nodig)

De specs schrijf je in het Nederlands; technische aanduidingen blijven in code-vorm. Verzin nooit een eigen, letterlijke woord-voor-woord-vertaling van een Engelse term — gebruik de Nederlandse term die het project zélf aan gebruikers toont.

- Kom je tijdens de migratie een houtige of onjuiste vertaling tegen en is er een codebase aanwezig → zoek de vertaalbestanden (i18n: `lang/`, `config/locales/`, `locales/`, `messages/nl.json`, etc.) en gebruik exact díe Nederlandse term.
- Geen codebase of geen vertaalbestand → kies een natuurlijke, gangbare Nederlandse term; bij twijfel behoud je de oorspronkelijke term.
- **Technische aanduidingen blijven in code-vorm** en vertaal je niet: modelnamen, attributen, enum-waarden en veldnamen in `Use cases` (Data), `Model Specs`, `Formuliervelden` (kolom Veld) en payload contracts.

---

## Stap 5 — Afronden

Geef na het wegschrijven een beknopt overzicht:

```
Migratie voltooid.

Bestand:   docs/specs/<domein>/<story>.md

Toegepast:
  - <categorie>: <korte beschrijving van wat is aangepast>
  - ...

Opgesplitst (indien van toepassing):
  - docs/specs/<domein>/<a>.md  (<story A>)
  - docs/specs/<domein>/<b>.md  (<story B>)

Ter controle (twijfelgevallen niet automatisch verwijderd):
  - <item>: <reden>
```

Laat secties weg die niet van toepassing zijn.
