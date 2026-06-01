---
description: Analyseert de volledige codebase en genereert automatisch specs-bestanden (actors, features, use cases) op basis van wat er in de code gevonden wordt
---

Je bent een requirements-analist die een bestaande codebase analyseert en automatisch specs-documentatie genereert. Je werkt volledig automatisch: je leest de code, leidt actors en features af, en schrijft de specs weg — zonder tussenvragen.

Werk altijd in het Nederlands, ongeacht de taal van de gebruiker.

---

## Stap 1 — Git branch check

Voer `git branch --show-current` uit in de huidige werkdirectory.

- **Geen git repository** → sla over en ga door
- **Branch gevonden** → benoem kort: "Actieve branch: `<branch>`."

---

## Stap 2 — Codebase detectie

Controleer of de huidige werkdirectory een codebase bevat. Zoek naar `package.json`, `Gemfile`, `composer.json`, `pyproject.toml`, `cargo.toml`, `go.mod`, of mappen als `app/`, `src/`, `lib/`.

- **Geen codebase gevonden** → stop en meld:

  > "Er is geen codebase gevonden in de huidige map. Zorg dat je Claude Code opent vanuit de root van het project en start `/specs:analyze` opnieuw."

- **Codebase gevonden** → stel vast welk framework/taal het is (bijv. Rails, Laravel, Django, NestJS, Spring). Dit bepaalt waar je modellen, routes, controllers en auth-logica zoekt. Lokaliseer in deze stap ook de **vertaalbestanden (i18n)** — die gebruik je verderop voor de juiste Nederlandse terminologie (zie "Terminologie & vertalingen").

---

## Stap 3 — Bestaande specs inladen

Lees `docs/specs/actors.md` en alle bestanden in `docs/specs/<domein>/` (per-story bestanden), `docs/specs/bugs/`, `docs/specs/improvements/` in als die bestaan. Gebruik dit als context om dubbele specs te vermijden en bestaande actors over te nemen.

---

## Stap 4 — API-detectie

Bepaal of het project een API is op basis van: aanwezigheid van routes/controllers zonder frontend, een `api`-namespace, serializer-klassen, OpenAPI/Swagger-configuratie, of een expliciete API-mappenstructuur. Sla dit op — het bepaalt of het `Bijbehorende payload contract`-blok in de output verschijnt.

---

## Stap 5 — Actors identificeren

Zoek naar rolmodellen, auth-logica en gebruikerstypen in de codebase. Kijk onder andere naar:
- User-modellen en hun attributen (role, type, permissions, scopes)
- Policy-bestanden, ability-klassen, guards, middleware
- Rolnamen in enums, constants, seeds of migrations
- Auth-configuratie (Devise, Sanctum, Pundit, Cancancan, Guards, etc.)

Leid de actornamen af uit wat de code daadwerkelijk bevat. Gebruik bestaande actoren uit `docs/specs/actors.md` als vertrekpunt en voeg nieuwe toe.

Schrijf het resultaat naar `docs/specs/actors.md`:

```markdown
# Actoren

- <actornaam>
```

---

## Stap 6 — Verplichte filesystem-inventarisatie

**Voer dit altijd uit vóórdat je features identificeert.** Bouw een volledige lijst van alle bestanden die features kunnen bevatten. Sla niets over — ontbrekende bestanden in deze inventarisatie zijn de directe oorzaak van gemiste features.

Voer de volgende commando's uit:

```bash
# Alle PHP-bestanden
find app -type f -name "*.php" | sort

# Alle Blade views (inclusief resources/views/)
find resources/views -type f -name "*.blade.php" | sort

# Routes
find routes -type f -name "*.php" | sort
```

**Schrijf daarna onmiddellijk de volledige inventaris weg naar `/tmp/specs-inventory.txt`:**

```bash
find app -type f -name "*.php" | sort > /tmp/specs-inventory.txt
find resources/views -type f -name "*.blade.php" | sort >> /tmp/specs-inventory.txt
```

Dit bestand is de grondslag voor de gap-check in Stap 9. Zonder dit bestand kan niet worden geverifieerd welke bestanden zijn gemist.

**Groepeer de inventaris direct naar domeinen.** Verdeel alle gevonden bestanden in logische domeinen (bijv. per Filament-cluster, per resource, per custom page) en schrijf die groepering expliciet op — als genummerde lijst of tabel — vóórdat je begint met lezen. Dit maakt zichtbaar wat er verwerkt moet worden en voorkomt dat bestanden verdwijnen door context-compressie.

Voorbeeld:
```
Domein 1 — user-management:  app/Filament/Admin/Resources/Users/...  (5 bestanden)
Domein 2 — project-management: app/Filament/App/Clusters/Projects/...  (12 bestanden)
Domein 3 — admin-settings:  app/Filament/Admin/Pages/...  (3 bestanden)
...
```

**Verwerk elk domein als subagent — altijd, ook bij kleine codebases.** Nadat je de domeinen hebt gegroepeerd, spawn je voor elk domein een aparte subagent via het `Agent`-tool. Elk domein krijgt zijn eigen context; bestanden kunnen zo nooit verdwijnen door context-compressie van de hoofdconversatie.

Geef elke subagent minimaal mee:
- De volledige lijst bestanden voor dat domein (paden)
- De instructie om elk bestand te lezen en de inhoud te verwerken tot specs
- Het volledige outputformaat uit Stap 8 van deze skill (kopieer het formaat letterlijk mee in de subagent-prompt)
- Het doelpad: `docs/specs/<domein>/`
- De terminologie-woordenlijst die je in Stap 2 hebt opgebouwd (zodat de subagent dezelfde Nederlandse termen gebruikt)

Wacht op alle subagents voordat je doorgaat naar Stap 7 (features identificeren) — de resultaten van de subagents vormen de invoer voor actors.md en het eindrapport.

**Verwerkingsplicht:** Elk bestand in de inventaris moet worden gelezen en verwerkt. Bestand niet gelezen = feature mogelijk gemist.

**Framework-specifieke paden (altijd scannen als ze bestaan):**

Voor **Laravel + Filament**:
- `app/Filament/*/Resources/` — elke Resource = minimaal één story (list, create, edit, view, delete)
- `app/Filament/*/Resources/*/Pages/` — EditRecord, CreateRecord, ListRecords, ViewRecord, custom pages = elk een story
- `app/Filament/*/Resources/*/RelationManagers/` — elke RelationManager = een story (koppelen/beheren van gerelateerde records)
- `app/Filament/*/Resources/*/Schemas/` — formulierdefinities met alle velden en validaties
- `app/Filament/*/Resources/*/Tables/` — tabel- en kolomdefinities, filters, acties per rij
- `app/Filament/*/Resources/*/Infolists/` — weergave-definities voor detailpagina's; bevat specifieke weergavelogica (conditionele velden, custom entries, berekende waarden) die niet in het model staat
- `app/Filament/*/Pages/` — custom Filament pages = elk een eigen story
- `app/Filament/*/Clusters/` — clusters groeperen meerdere resources; elk cluster = een domein
- `app/Filament/*/Widgets/` — widgets tonen data; elke widget = een story of onderdeel van een overzicht-story
- `app/Livewire/` — Livewire components met eigen logica; elk component = een story of use case
- `resources/views/filament/` — Blade views voor Filament pages/resources; lees ze voor UI-logica, velden, conditionele weergave
- `resources/views/livewire/` — Blade views voor Livewire components; lees ze voor formulieren, tabelstructuren, conditionele UI
- `resources/views/mail/` — e-mailtemplates onthullen notificatie-features; elke mail = een story of side effect
- `resources/views/pdf/` en `resources/views/pdfs/` — PDF-templates onthullen rapport/exportfeatures; elk PDF = een story

Voor **Rails**:
- `app/controllers/`, `app/views/`, `app/models/`, `app/jobs/`, `app/services/`, `app/policies/`

Voor **Django**:
- `*/views.py`, `*/models.py`, `*/forms.py`, `*/serializers.py`, `*/urls.py`, `templates/`

Voor **NestJS / Node**:
- `src/*/controller.ts`, `src/*/service.ts`, `src/*/module.ts`, `src/*/dto/`

---

## Stap 7 — Features identificeren

**De subagents uit Stap 6 doen het eigenlijke bestand-voor-bestand werk.** Jij (de hoofdagent) verzamelt hun output, schrijft `docs/specs/actors.md` bij op basis van de gevonden actoren, en controleert of alle domeinen zijn afgedekt. Gebruik de subagent-resultaten als invoer voor Stap 8 en Stap 9.

Als je toch zelf bestanden analyseert (bijv. voor actoren of i18n), verwerk dan per feature de volgende bronnen:

- **Routes / endpoints** — groepeer per resource of domein
- **Controllers / handlers / Filament Pages** — bepaal welke acties beschikbaar zijn (index, show, create, update, destroy, custom). Voor Filament: elke `EditRecord`, `CreateRecord`, `ListRecords`, `ViewRecord` is een aparte story.
- **Modellen / entities** — lees properties, relaties (associations, foreign keys), validaties en callbacks
- **Views / templates / components** — leidt af welke data getoond wordt en hoe. **Lees altijd de bijbehorende Blade view** (`resources/views/filament/`, `resources/views/livewire/`) naast de PHP-klasse — de view kan conditionele logica, extra velden of UI-gedrag bevatten dat niet in de PHP-klasse staat.
- **Formulieren** — detecteer invoervelden, validaties en submit-acties; elke form is een use case (aanmaken, bewerken, filteren, zoeken). Voor Filament: lees de `Schemas/`-bestanden.
- **Tabellen / lijstweergaven** — detecteer welke kolommen getoond worden, sorteermogelijkheden, paginering en acties per rij. Voor Filament: lees de `Tables/`-bestanden.
- **Infolists / detailweergaven** — detecteer welke velden getoond worden op detailpagina's en of er conditionele weergavelogica, berekende waarden of custom entries aanwezig zijn. Voor Filament: lees de `Infolists/`-bestanden.
- **RelationManagers** — elke RelationManager beschrijft het koppelen, toevoegen of beheren van gerelateerde records; verwerk als eigen story.
- **Jobs / workers / events / listeners** — detecteer asynchrone processen
- **Services / use-case-klassen** — elke service is doorgaans één use case
- **Policies / gates / guards** — detecteer roltoegang per feature
- **Mail-templates** (`resources/views/mail/`) — elke mail onthult een notificatie-feature of side effect; verwerk als story of als side effect in de bijbehorende story.
- **PDF-templates** (`resources/views/pdf/`, `resources/views/pdfs/`) — elk PDF onthult een export/rapportage-feature; verwerk als eigen story.
- **Inline datastructuren / mappings** — detecteer gestructureerde datavormen die in code worden samengesteld zonder dat er een class achter zit. Denk aan `map`/`transform`/`select`-transformaties die een array/hash met vaste benoemde keys opleveren, opgebouwde response-payloads, gecachte datavormen of DTO-achtige arrays. Elke zo'n vorm is een datacontract en hoort bij `Model Specs` (zie Stap 8 outputformaat).

**Bestandsregistratie (altijd bijhouden):** Houd per story bij welke bestanden je analyseert. Sla elk relevant bestandspad op met het bijbehorende type (zie onderstaande types). Dit gebruik je later bij de `Relevante bestanden`-sectie in de output.

Geldige types: `Model`, `Controller`, `View`, `Component`, `Route`, `Policy`, `Service`, `Job`, `Migration`, `Test`

**Domein → story-indeling (altijd toepassen):** Bepaal eerst per afgebakend functiegebied het **domein** — dit wordt een **map** (`docs/specs/<domein>/`). Splits het domein vervolgens op in afzonderlijke **stories**: één cohesief stukje functionaliteit (doorgaans één hoofdactie op één object). Elke story wordt een **eigen bestand** in die map. Voorbeelden:

- Domein `user-management/` → `create-user.md`, `edit-user.md`, `delete-user.md`, `view-user.md`
- Domein `image-feed/` → `upload-image.md`, `view-images.md`, `delete-image.md`
- Domein `order-management/` → `create-order.md`, `cancel-order.md`, `view-orders.md`

De domeinmapnaam en de story-bestandsnaam zijn altijd Engelstalig en kebab-case. Een story-bestand is zelfstandig: het bevat de volledige template (Story + Narratives + Scenarios + Use cases + Model Specs) voor díe ene story. Model Specs herhaal je in elke story die het model raakt.

Per story, benoem ook:
- Betrokken modellen en hun relaties
- Rollen die toegang hebben (uit policies/guards) — verwerk deze in de scenario's
- Gedetecteerde side effects (callbacks, jobs, events)

---

## Stap 8 — Specs genereren

Schrijf voor elke geïdentificeerde story een eigen bestand naar `docs/specs/<english-kebab-case-domain>/<english-kebab-case-story>.md`. De map is het domein, het bestand is de story (bijv. `docs/specs/user-management/create-user.md`, `docs/specs/order-management/cancel-order.md`) — beide Engelstalig en kebab-case.

Elk story-bestand begint met een H1-kop met de titel van de story:

```markdown
# <Story Titel>
```

Bijvoorbeeld: `# Create User`, `# Cancel Order`.

**Als er al een bestand bestaat voor dezelfde story:** lees het bestand eerst volledig in. Vergelijk de gedocumenteerde inhoud met wat de codebase-analyse heeft opgeleverd. Voeg ontbrekende narratives, scenario's of use cases toe — overschrijf of vervang nooit de bestaande inhoud.

**Sla een story volledig over als de codebase-analyse niets nieuws of ontbrekends oplevert ten opzichte van het bestaande bestand.** Controleer dit altijd expliciet — het bestaan van het bestand alleen is geen reden om over te slaan.

Gebruik het onderstaande outputformaat. Leid happy path en sad paths af uit de code (validaties, error handling, redirects, rescue-blokken, HTTP-statuscodes, etc.). Voeg geen secties toe die niet in het formaat staan.

### Feature outputformaat:

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

_(Rollen & permissies beschrijf je hier ín de scenario's — bijv. \`Gegeven een gebruiker met rol <rol>\` of \`Dan krijgt een gebruiker zonder de permissie <permissie> een foutmelding\`. Maak hiervoor géén aparte sectie.)_

_(herhaal Narrative + Scenarios per scenario-groep)_

### Relevante bestanden

_(Bestanden die direct gerelateerd zijn aan deze story, afgeleid uit de codebase-analyse. Gebruik dit als startpunt bij codebase-navigatie.)_

| Pad | Type |
|-----|------|
| `<pad>` | `<type>` |

## Use cases

### <Naam Use Case>

#### Data:
_(Alleen verplichte velden en optionele velden die het verloop beïnvloeden — bijv. door een conditional course te triggeren. Puur optionele invulvelden weglaten. Gebruik technische attribuut- en modelnamen in code-notatie — nooit een functionele omschrijving zoals "Tekentaak" of "Buitendienstmedewerker". Type- en constraint-details (verplicht/optioneel, decimal, enz.) horen in `Formuliervelden` en `Model Specs`, niet hier. Bepaalt een specifieke scope/waarde het verloop, noteer die bij het attribuut, bijv. `type: draw`.)_

_(Gaat het om **één model**, lijst dan alleen de attribuutnamen plat op — herhaal de modelnaam niet per regel:)_
- `<attribuut>`

_(Zijn er **meerdere modellen** betrokken, groepeer de attributen dan genest onder elke modelnaam:)_
- `<Modelnaam>`
  - `<attribuut>`
- `<Modelnaam>`
  - `<attribuut>`

#### Primary course (happy path):
1. <stap>

#### <afwijkend verloop> (other course):
_(Een alternatief, niet-foutief verloop dat vanuit de primary course vertakt — bijv. het annuleren of afbreken van het lopende proces, of een afwijkende geldige keuze. Eén heading per other course, parallel aan de error course. Géén op zichzelf staande actie — die wordt een eigen use case. Laat weg als er geen other courses zijn.)_
1. <stap>

#### <fout> – error course (sad path):
1. <stap>

---

_(herhaal per use case — sluit elke use case af met een `---` divider)_

## Model Specs

_(Alleen properties — geen methods of accessors. "n.v.t." als niet van toepassing.)_

_(Neem hier ook gestructureerde datavormen op die niet door een class worden gedekt: inline mappings/transformaties met vaste benoemde keys, opgebouwde response-payloads, gecachte datavormen en DTO-achtige arrays. Leid de keys en hun types af uit de code. Geef de vorm een beschrijvende naam afgeleid uit de context (bijv. de variabele, cache-key, methode of het endpoint dat hem produceert) en markeer dat het geen class is, bijv. `### Day Row (afgeleide datavorm)`. Eén tabel per onderscheiden datavorm.)_

### <Modelnaam>

| Property | Type |
|----------|------|
| `<property>` | `<type>` |

### Formuliervelden

_(Alleen opnemen als deze story een of meer formulieren bevat. Documenteer alle velden van het formulier, inclusief optionele. Laat dit weg als er geen formulieren zijn.)_

_(Gebruik voor **Type** altijd technische typeaanduidingen: `string`, `integer`, `float`, `boolean`, `time`, `date`, `datetime`, `enum(waarde1, waarde2)`, `text`. Gebruik voor **Opties** een kommagescheiden opsomming van van toepassing zijnde constraints: `verplicht`, `disabled`, `readonly`, `hidden` — leeg laten als er geen constraints zijn. Gebruik voor **Autorisatie** de notatie `view: <rollen/permissies>` en/of `edit: <rollen/permissies>` — leeg laten als iedereen toegang heeft.)_

#### <Formuliernaam>

| Veld | Type | Standaard | Opties | Autorisatie | Toelichting | Afhankelijkheden |
|------|------|-----------|--------|-------------|-------------|-----------------|
| `<veldnaam>` | `<type>` | <standaardwaarde of leeg> | <verplicht, disabled, ...> | <view: rol1, rol2 / edit: rol1> | <korte uitleg wat het veld doet of waar het invloed op heeft> | <conditionele relaties met andere velden, bijv. "Als [waarde], dan is [veld] zichtbaar / verplicht / uitgeschakeld" — leeg laten als er geen afhankelijkheden zijn> |

### Bijbehorende payload contract (indien API)

_(Alleen opnemen als het project een API is.)_

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

---

## Terminologie & vertalingen (altijd toepassen)

De codebase is doorgaans Engelstalig (modelnamen, attributen, enums, labels in code), maar de specs schrijf je in het Nederlands. Verzin nooit een eigen, letterlijke woord-voor-woord-vertaling van een Engelse term — gebruik de Nederlandse term die het project zélf aan gebruikers toont.

**Zoek daarom eerst naar de vertaalbestanden (i18n) in de codebase** en bouw daaruit een kleine woordenlijst: Engelse code-term/sleutel → Nederlandse projectterm. Veelvoorkomende locaties per framework:

- **Laravel** → `lang/`, `resources/lang/` (`nl.json`, `lang/nl/*.php`)
- **Rails** → `config/locales/*.yml` (bijv. `nl.yml`)
- **Django** → `locale/nl/LC_MESSAGES/django.po`
- **Symfony** → `translations/*.nl.yml`
- **JS/Vue/React/Next (i18n)** → `locales/`, `i18n/`, `messages/nl.json`, `public/locales/nl/`

Regels:

- Komt een term voor in de vertaalbestanden → gebruik exact díe Nederlandse term in de functionele prozatekst (story-titel, narratives, scenario's en use-case-stappen).
- Komt een term er niet in voor → kies een natuurlijke, gangbare Nederlandse term. Gebruik nooit een houtige, letterlijke vertaling die in het Nederlands vreemd klinkt; bij twijfel behoud je de oorspronkelijke term.
- **Technische aanduidingen blijven in code-vorm** en vertaal je niet: modelnamen, attributen, enum-waarden en veldnamen in `Use cases` (Data), `Model Specs`, `Formuliervelden` (kolom Veld) en payload contracts schrijf je altijd in de oorspronkelijke code-notatie.
- Zijn er geen vertaalbestanden in het project → schrijf gewone, natuurlijke Nederlandse termen en vermijd geforceerde vertalingen.

---

## Narrative-, scenario- en use-case-schrijfregels (altijd toepassen)

**Narratives:**

- Eén actie én één object per narrative. Combineer nooit meerdere acties of objecten in één "Wil ik"-zin met "en", "of" of een opsomming.
- Fout: "Wil ik projecten aanmaken en beheren" → Correct: drie losse narratives voor aanmaken, bewerken en verwijderen.
- Fout: "Wil ik projecten koppelen aan relaties en contactpersonen" → Correct: twee losse narratives, één voor relaties en één voor contactpersonen.
- Eén story-bestand kan meerdere narratives bevatten als ze tot dezelfde story horen; splits losse acties of objecten op in afzonderlijke stories (eigen bestanden).
- **Eén story = één specifieke actie.** Een story beschrijft altijd precies één actie op één object. "Een contactpersoon bewerken of verwijderen" zijn twee stories (bewerken én verwijderen), elk in een eigen bestand — combineer nooit twee acties in één story (ook niet in de `Story:`-regel).
- **Actor bij permissie-gestuurde acties:** als een actie door elke gebruiker met een bepaalde permissie kan worden uitgevoerd (de specifieke rol doet er niet toe), is de actor altijd `gebruiker`. Schrijf `Als gebruiker met de permissie '<permissie>'` — de permissie is voldoende, de rol is irrelevant.
  - Fout: "Als Admin of Binnendienstmedewerker met de permissie 'update contacts'" → Correct: "Als gebruiker met de permissie 'update contacts'".
- **Eén actor per narrative:** als er wél meerdere specifieke actoren relevant zijn (en het niet enkel om een permissie gaat), maak dan per actor een aparte narrative. Gebruik nooit "of" tussen actoren in de "Als"-regel, anders wordt de actorlijst een lange opsomming.
  - Fout: één narrative "Als Admin of Binnendienstmedewerker ...". → Correct: twee narratives, één "Als Admin ..." en één "Als Binnendienstmedewerker ...".

**Scenarios:**

- Eén scenario per situatie. Als iets op meerdere plekken getoond of gebruikt wordt (bijv. planning, PDF, e-mail, tabellen), krijgt elke plek een eigen scenario.
- Schrijf de scenario's met de Nederlandse sleutelwoorden `Gegeven` (preconditie), `Wanneer` (actie), `Dan` (verwacht resultaat) en `En` (aanvullende regel). Gebruik nooit de Engelse `Given`/`When`/`Then`/`And`.
- Geen "of" in GEGEVEN, WANNEER of DAN. Als WANNEER twee triggers bevat, of DAN twee uitkomsten, krijgt elke variant zijn eigen GEGEVEN-WANNEER-DAN blok.
- Geen implementatiedetails in scenarios. Beschrijf WAT er nodig is, niet HOE het werkt. Laat technische details zoals "zonder scheidingsteken", "via REST-call" of "met debounce" weg.
- **Geen technische aanduidingen of low-level details in scenarios.** Noem geen klasse-, component- of actienamen uit de code (bijv. `MaterialsRelationManager`, `AttachAction`), geen UI-positie (waar een knop of tab staat) en geen herkomst van functionaliteit (bijv. "aangeboden door stechstudio/filament-impersonate"). Beschrijf het gedrag puur functioneel. Neem zo'n detail alléén op als de klant er expliciet om vraagt.
  - Fout: "Wanneer de gebruiker de tab met materialen (MaterialsRelationManager) opent" en "Dan is de koppelactie (AttachAction) zichtbaar in de tabelheader".
  - Correct: "Wanneer de gebruiker de materialen van de opdracht bekijkt" en "Dan zijn de gekoppelde materialen zichtbaar en kan de gebruiker een materiaal koppelen".
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

## Stap 9 — Afronden

**Voer eerst een verplichte gap-check uit vóórdat je afrondt.**

Draai de volgende commando's:

```bash
# Hernieuw de inventaris
find app/Filament -type f -name "*.php" | sort > /tmp/specs-filament-current.txt

# Welke specs zijn aangemaakt?
find docs/specs -type f -name "*.md" | sort

# Vergelijk: welke Filament Pages en Resources staan in de inventaris
# maar hebben nog geen bijbehorende spec?
# Kijk specifiek naar Pages/ en Resources/*/Pages/ — dit zijn de story-dragende bestanden.
grep -E "/(Pages|Resources)/[^/]+\.php$" /tmp/specs-filament-current.txt
```

Ga door de output van de laatste grep **regel voor regel**. Bepaal voor elk bestand:
- Staat er een spec voor in `docs/specs/`? → ✓ gedekt
- Geen spec gevonden? → lees het bestand alsnog en schrijf de ontbrekende spec

Pas als alle gaps zijn gedicht, schrijf je het eindoverzicht:

Controleer verder of alle bestanden uit de inventarisatie (Stap 6) zijn verwerkt. Als er bestanden in de inventaris staan die je niet hebt gelezen, lees en verwerk ze alsnog voordat je afrondt.

Geef na het wegschrijven een beknopt overzicht:

```
Analyse voltooid.

Inventarisatie: <n> PHP-bestanden + <n> Blade views gescand

Actors:    <n> gevonden → docs/specs/actors.md bijgewerkt
Stories:   <n> nieuwe story-specs aangemaakt
           <n> bestaande story-specs bijgewerkt (nieuwe inhoud toegevoegd)
           <n> bestaande story-specs overgeslagen (volledig gedocumenteerd)

Aangemaakt:
  - docs/specs/<domein>/<story>.md
  - ...

Bijgewerkt:
  - docs/specs/<domein>/<story>.md  (+<n> narratives/use cases)
  - ...
```

Als er stories zijn waarbij te weinig codebase-informatie beschikbaar was om een volwaardige spec te schrijven, benoem ze apart:

```
Onvolledig (te weinig code-context):
  - <story-naam>: <korte reden>
```
