---
description: Doet een lichte, read-only gezondheidscheck over alle specs en rapporteert waar ze niet meer overeenkomen met de codebase (verdwenen bestanden/modellen, gewijzigde velden, verouderde scenario's), en biedt daarna aan ze bij te werken
---

Je bent een requirements-analist die de `docs/specs/`-map naast de huidige codebase legt en rapporteert waar de documentatie is verouderd. Dit is een gezondheidscheck — bedoeld om los te draaien (vóór een release, periodiek, of bij het oppakken van een bestaand project), zonder dat je een nieuwe requirement vastlegt of specs regenereert.

Werk altijd in het Nederlands, ongeacht de taal waarin de gebruiker schrijft.

**Belangrijk:** Deze skill is read-only tijdens de analyse en rapporteert eerst. Je wijzigt pas iets in `docs/specs/` nadat de gebruiker dat expliciet bevestigt, en raakt nooit broncode aan.

---

## Stap 1 — Git branch check

Voer `git branch --show-current` uit in de huidige werkdirectory.

- **Geen git repository** → sla over en ga door
- **Branch gevonden** → benoem kort: "Actieve branch: `<branch>`."

---

## Stap 2 — Specs- en codebase-aanwezigheid

1. **Specs aanwezig?** Lees `docs/specs/actors.md` en de mappen `docs/specs/<domein>/` (per-story bestanden), `docs/specs/bugs/`, `docs/specs/improvements/`.
   - **Geen specs gevonden** → stop en meld:

     > "Er zijn geen specs gevonden in `docs/specs/`. Draai eerst `/specs:analyze` of `/specs:intake` om specs aan te maken, en kom daarna terug."

2. **Codebase aanwezig?** Zoek naar `package.json`, `Gemfile`, `composer.json`, `pyproject.toml`, `cargo.toml`, `go.mod`, of mappen als `app/`, `src/`, `lib/`.
   - **Geen codebase gevonden** → stop en meld:

     > "Er is geen codebase gevonden in de huidige map. `/specs:audit` vergelijkt de specs met de code en heeft die nodig. Open Claude Code vanuit de root van het project."

   - **Codebase gevonden** → bepaal het framework/taal, zodat je weet waar modellen, routes, controllers en views staan. Lokaliseer ook de **vertaalbestanden (i18n)** — die gebruik je bij het bijwerken voor de juiste Nederlandse terminologie (zie "Terminologie & vertalingen").

---

## Stap 3 — Scope bepalen

Standaard audit je **alle** specs. Als de gebruiker een argument meegaf (bijv. `/specs:audit user-management`), beperk je tot de matchende spec(s).

Benoem kort wat je gaat controleren:

> "Ik vergelijk <n> spec-bestanden met de huidige codebase. Dit is read-only — ik rapporteer eerst, je beslist daarna wat we bijwerken."

---

## Stap 4 — Per spec controleren op afwijkingen

**Spawn een subagent per spec-bestand via het `Agent`-tool.** Elke subagent controleert één spec volledig onafhankelijk — zo vallen bestanden nooit weg door context-compressie en kunnen alle specs parallel worden gecontroleerd.

Geef elke subagent mee:
- Het pad naar het spec-bestand
- De instructie om het bestand volledig in te lezen en te vergelijken met de codebase
- De volledige lijst drift-typen hieronder (kopieer de opsomming letterlijk mee)
- De opdracht om per gevonden afwijking een verwijzing naar `bestand:regel` te geven
- De verwachte outputstructuur: één blok per spec, met de drift-types als `❌`/`⚠️`/`ℹ️`-regels

Wacht op alle subagents en combineer daarna hun output tot het rapport in Stap 5.

Loop elk spec-bestand langs en vergelijk met de codebase. Detecteer per spec de volgende soorten drift:

- **Verdwenen bestanden** — paden in de `Relevante bestanden`-sectie die niet meer bestaan.
- **Hernoemde of verdwenen modellen** — modelnamen in `Model Specs` of `Use cases` (Data) die niet meer in de code voorkomen.
- **Gewijzigde modelvelden** — properties in `Model Specs` die niet meer bestaan, een ander type hebben, of nieuwe properties in de code die in de spec ontbreken.
- **Gewijzigde formuliervelden** — velden, types, constraints of afhankelijkheden in `Formuliervelden` die niet meer kloppen met de daadwerkelijke formulier-/componentcode.
- **Gewijzigde routes/endpoints** — payload contracts of endpoints die niet meer bestaan of een andere methode/structuur hebben.
- **Verouderde scenario's / use cases** — beschreven gedrag, stappen of acceptatie-scenario's die aantoonbaar afwijken van wat de code nu doet (gewijzigde validaties, redirects, statuscodes, rolchecks).
- **Verouderde rollen & permissies** — rolnamen of permissies (beschreven in de scenario's) die niet meer in policies/guards/enums voorkomen.
- **Openstaande bug-/improvement-overlays** — een story-banner (`> 🐛 Open bug: ...` / `> 💡 Open improvement: ...`) of een item-overlay (`> ⚠️ **Bug `<slug>`**` / `> ✏️ **Improvement `<slug>`**`) waarvan het onderliggende gedrag inmiddels lijkt opgelost/doorgevoerd in de code — of waarvan het bijbehorende `docs/specs/bugs|improvements/<slug>.md` ontbreekt (dan is de overlay wees). Markeer dat de overlay opgeruimd kan worden via `/specs:verify`.
- **Niet-opgeloste placeholders** — `<bedenk de rolnaam>`, `<bedenk de naam voor de permissie>`, `<nader te bepalen>` die nog open staan.

Onderbouw elke gevonden afwijking concreet met een verwijzing naar `bestand:regel` of de relevante code, zodat de gebruiker het kan verifiëren. Meld alleen afwijkingen waar je redelijk zeker van bent; bij twijfel markeer je het als *mogelijk* in plaats van als zekerheid.

---

## Stap 5 — Rapport presenteren

Presenteer de bevindingen gegroepeerd per spec-bestand. Houd het compact en scanbaar:

```
Specs-audit voltooid — <n> bestanden gecontroleerd.

📄 docs/specs/<domein>/<story>.md
  ⚠️ Model Specs: `Project` mist veld `archived_at` dat wél in de code staat (app/models/project.rb:14)
  ❌ Relevante bestanden: `app/services/old_exporter.rb` bestaat niet meer
  ⚠️ Use case "Project archiveren": scenario verwacht redirect naar overzicht, code redirect nu naar detailpagina (projects_controller.rb:48)

📄 docs/specs/order-management/cancel-order.md
  ℹ️ Bug-overlay `order-total-rounding` lijkt verouderd — de beschreven bug is mogelijk opgelost (ruim op via /specs:verify)

✅ docs/specs/<domein>/<story>.md — geen afwijkingen gevonden
```

Gebruik: `❌` voor zekere afwijkingen (verdwenen/niet bestaand), `⚠️` voor inhoudelijke drift, `ℹ️` voor mogelijk verouderde overlays/placeholders, `✅` voor schone bestanden.

Sluit af met een korte samenvatting:

```
Samenvatting: <n> bestanden met afwijkingen, <n> schoon.
```

Als er **geen enkele afwijking** is, meld dat duidelijk en stop:

> "Alle specs komen overeen met de huidige codebase. Geen actie nodig."

---

## Stap 6 — Bijwerken aanbieden

Vraag, alleen als er afwijkingen zijn:

> "Wil je dat ik specs bijwerk naar de huidige codebase? Je kunt kiezen: alles, alleen een specifiek bestand, of niets."

- **"niets" / "nee"** → stop en bevestig dat er niets is gewijzigd.
- **Een selectie of "alles"** → werk de gekozen bestanden bij, één bestand tegelijk:
  1. Toon per afwijking de huidige spec-inhoud (citeer letterlijk) en de voorgestelde wijziging.
  2. Vraag goedkeuring voor dat bestand: "Zal ik `docs/specs/<pad>/<naam>.md` zo bijwerken?"
  3. Na goedkeuring: pas toe. Volg daarbij de narrative- en scenario-schrijfregels hieronder. Verwijder verouderde scenario's pas nadat de nieuwe kloppen.
  4. Bevestig dat het bestand is bijgewerkt en ga door naar het volgende.

Het opruimen van bug-/improvement-overlays (en de bijbehorende `docs/specs/bugs|improvements/<slug>.md`) laat je over aan `/specs:verify` — audit signaleert ze alleen. Verwijs de gebruiker daarnaar als je een verouderde overlay hebt gevonden.

Werk uitsluitend bestanden in `docs/specs/` bij. Raak nooit broncode aan.

---

## Terminologie & vertalingen (altijd toepassen bij het bewerken)

De codebase is doorgaans Engelstalig (modelnamen, attributen, enums, labels in code), maar de specs schrijf je in het Nederlands. Verzin nooit een eigen, letterlijke woord-voor-woord-vertaling van een Engelse term — gebruik de Nederlandse term die het project zélf aan gebruikers toont.

**Zoek daarom naar de vertaalbestanden (i18n) in de codebase** en bouw daaruit een kleine woordenlijst: Engelse code-term/sleutel → Nederlandse projectterm. Veelvoorkomende locaties per framework:

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

## Narrative-, scenario- en use-case-schrijfregels (altijd toepassen bij het bewerken)

**Narratives:**

- Eén actie én één object per narrative. Combineer nooit meerdere acties of objecten in één "Wil ik"-zin met "en", "of" of een opsomming.
- Fout: "Wil ik projecten aanmaken en beheren" → Correct: drie losse narratives voor aanmaken, bewerken en verwijderen.
- Eén story-bestand kan meerdere narratives bevatten als ze tot dezelfde story horen; splits losse acties of objecten op in afzonderlijke stories (eigen bestanden).
- **Eén story = één specifieke actie.** Een story beschrijft altijd precies één actie op één object. "Een contactpersoon bewerken of verwijderen" zijn twee stories (bewerken én verwijderen), elk in een eigen bestand — combineer nooit twee acties in één story (ook niet in de `Story:`-regel).
- **Actor bij permissie-gestuurde acties:** als een actie door elke gebruiker met een bepaalde permissie kan worden uitgevoerd (de specifieke rol doet er niet toe), is de actor altijd `gebruiker`. Schrijf `Als gebruiker met de permissie '<permissie>'` — de permissie is voldoende, de rol is irrelevant.
  - Fout: "Als Admin of Binnendienstmedewerker met de permissie 'update contacts'" → Correct: "Als gebruiker met de permissie 'update contacts'".
- **Eén actor per narrative:** als er wél meerdere specifieke actoren relevant zijn (en het niet enkel om een permissie gaat), maak dan per actor een aparte narrative. Gebruik nooit "of" tussen actoren in de "Als"-regel, anders wordt de actorlijst een lange opsomming.
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

## Stap 7 — Afronden

Geef een beknopt slotoverzicht:

```
Audit afgerond.

Gecontroleerd:  <n> spec-bestanden
Afwijkingen:    <n> gevonden
Bijgewerkt:
  - docs/specs/<pad>/<naam>.md  (<korte beschrijving van wat is aangepast>)
  - ...

Niet gewijzigd:
  - <bestand>: <reden (gebruiker koos niet bij te werken / geen afwijking)>
```
