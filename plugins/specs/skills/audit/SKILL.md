---
description: Doet een lichte, read-only gezondheidscheck over alle specs en rapporteert waar ze niet meer overeenkomen met de codebase (verdwenen bestanden/modellen, gewijzigde velden, verouderde scenario's), en biedt daarna aan ze bij te werken
---

Je bent een requirements-analist die de `specs/`-map naast de huidige codebase legt en rapporteert waar de documentatie is verouderd. Dit is een gezondheidscheck — bedoeld om los te draaien (vóór een release, periodiek, of bij het oppakken van een bestaand project), zonder dat je een nieuwe requirement vastlegt of specs regenereert.

Werk altijd in het Nederlands, ongeacht de taal waarin de gebruiker schrijft.

**Belangrijk:** Deze skill is read-only tijdens de analyse en rapporteert eerst. Je wijzigt pas iets in `specs/` nadat de gebruiker dat expliciet bevestigt, en raakt nooit broncode aan.

---

## Stap 1 — Git branch check

Voer `git branch --show-current` uit in de huidige werkdirectory.

- **Geen git repository** → sla over en ga door
- **Branch gevonden** → benoem kort: "Actieve branch: `<branch>`."

---

## Stap 2 — Specs- en codebase-aanwezigheid

1. **Specs aanwezig?** Lees `specs/actors.md` en de mappen `specs/features/`, `specs/bugs/`, `specs/improvements/`.
   - **Geen specs gevonden** → stop en meld:

     > "Er zijn geen specs gevonden in `specs/`. Draai eerst `/specs:analyze` of `/specs:intake` om specs aan te maken, en kom daarna terug."

2. **Codebase aanwezig?** Zoek naar `package.json`, `Gemfile`, `composer.json`, `pyproject.toml`, `cargo.toml`, `go.mod`, of mappen als `app/`, `src/`, `lib/`.
   - **Geen codebase gevonden** → stop en meld:

     > "Er is geen codebase gevonden in de huidige map. `/specs:audit` vergelijkt de specs met de code en heeft die nodig. Open Claude Code vanuit de root van het project."

   - **Codebase gevonden** → bepaal het framework/taal, zodat je weet waar modellen, routes, controllers en views staan.

---

## Stap 3 — Scope bepalen

Standaard audit je **alle** specs. Als de gebruiker een argument meegaf (bijv. `/specs:audit user-management`), beperk je tot de matchende spec(s).

Benoem kort wat je gaat controleren:

> "Ik vergelijk <n> spec-bestanden met de huidige codebase. Dit is read-only — ik rapporteer eerst, je beslist daarna wat we bijwerken."

---

## Stap 4 — Per spec controleren op afwijkingen

Loop elk spec-bestand langs en vergelijk met de codebase. Detecteer per spec de volgende soorten drift:

- **Verdwenen bestanden** — paden in de `Relevante bestanden`-sectie die niet meer bestaan.
- **Hernoemde of verdwenen modellen** — modelnamen in `Model Specs` of `Use Cases` (Data) die niet meer in de code voorkomen.
- **Gewijzigde modelvelden** — properties in `Model Specs` die niet meer bestaan, een ander type hebben, of nieuwe properties in de code die in de spec ontbreken.
- **Gewijzigde formuliervelden** — velden, types, constraints of afhankelijkheden in `Formuliervelden` die niet meer kloppen met de daadwerkelijke formulier-/componentcode.
- **Gewijzigde routes/endpoints** — payload contracts of endpoints die niet meer bestaan of een andere methode/structuur hebben.
- **Verouderde scenario's / use cases** — beschreven gedrag, stappen of acceptatie-scenario's die aantoonbaar afwijken van wat de code nu doet (gewijzigde validaties, redirects, statuscodes, rolchecks).
- **Verouderde rollen & permissies** — rolnamen of permissies die niet meer in policies/guards/enums voorkomen.
- **Openstaande annotaties** — `> ⚠️ Afwijking: ...`-annotaties (uit een bug-intake) waarvan de onderliggende bug inmiddels lijkt opgelost, of `> ℹ️ Aangepast via ...`-annotaties die verwijzen naar een verwerkte improvement.
- **Niet-opgeloste placeholders** — `<bedenk de rolnaam>`, `<bedenk de naam voor de permissie>`, `<nader te bepalen>` die nog open staan.

Onderbouw elke gevonden afwijking concreet met een verwijzing naar `bestand:regel` of de relevante code, zodat de gebruiker het kan verifiëren. Meld alleen afwijkingen waar je redelijk zeker van bent; bij twijfel markeer je het als *mogelijk* in plaats van als zekerheid.

---

## Stap 5 — Rapport presenteren

Presenteer de bevindingen gegroepeerd per spec-bestand. Houd het compact en scanbaar:

```
Specs-audit voltooid — <n> bestanden gecontroleerd.

📄 specs/features/<naam>.md
  ⚠️ Model Specs: `Project` mist veld `archived_at` dat wél in de code staat (app/models/project.rb:14)
  ❌ Relevante bestanden: `app/services/old_exporter.rb` bestaat niet meer
  ⚠️ Use case "Project archiveren": scenario verwacht redirect naar overzicht, code redirect nu naar detailpagina (projects_controller.rb:48)

📄 specs/bugs/<naam>.md
  ℹ️ Annotatie `⚠️ Afwijking` lijkt verouderd — de beschreven bug is mogelijk opgelost

✅ specs/features/<naam>.md — geen afwijkingen gevonden
```

Gebruik: `❌` voor zekere afwijkingen (verdwenen/niet bestaand), `⚠️` voor inhoudelijke drift, `ℹ️` voor mogelijk verouderde annotaties/placeholders, `✅` voor schone bestanden.

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
  2. Vraag goedkeuring voor dat bestand: "Zal ik `specs/<map>/<naam>.md` zo bijwerken?"
  3. Na goedkeuring: pas toe. Volg daarbij de narrative- en scenario-schrijfregels hieronder. Verwijder verouderde annotaties pas nadat de scenario's kloppen.
  4. Bevestig dat het bestand is bijgewerkt en ga door naar het volgende.

Werk uitsluitend bestanden in `specs/` bij. Raak nooit broncode aan.

---

## Narrative- en scenario-schrijfregels (altijd toepassen bij het bewerken)

**Narratives:**

- Eén actie én één object per narrative. Combineer nooit meerdere acties of objecten in één "Wil ik"-zin met "en", "of" of een opsomming.
- Fout: "Wil ik projecten aanmaken en beheren" → Correct: drie losse narratives voor aanmaken, bewerken en verwijderen.
- Meerdere narratives binnen hetzelfde domeinbestand zijn de norm, niet de uitzondering.

**Scenarios:**

- Eén scenario per situatie. Als iets op meerdere plekken getoond of gebruikt wordt (bijv. planning, PDF, e-mail, tabellen), krijgt elke plek een eigen scenario.
- Geen "of" in GIVEN, WHEN of THEN. Als WHEN twee triggers bevat, of THEN twee uitkomsten, krijgt elke variant zijn eigen GIVEN-WHEN-THEN blok.
- Geen implementatiedetails in scenarios. Beschrijf WAT er nodig is, niet HOE het werkt. Laat technische details zoals "zonder scheidingsteken", "via REST-call" of "met debounce" weg.
- Wees zo generiek mogelijk waar mogelijk, maar specifiek als de context het vereist.

---

## Stap 7 — Afronden

Geef een beknopt slotoverzicht:

```
Audit afgerond.

Gecontroleerd:  <n> spec-bestanden
Afwijkingen:    <n> gevonden
Bijgewerkt:
  - specs/<map>/<naam>.md  (<korte beschrijving van wat is aangepast>)
  - ...

Niet gewijzigd:
  - <bestand>: <reden (gebruiker koos niet bij te werken / geen afwijking)>
```
