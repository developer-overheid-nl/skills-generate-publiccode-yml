---
name: generate-publiccode-yml
description: |
Generate or update a publiccode.yml file for a Dutch government open source project. 
Use when the user wants to create or update publiccode.yml in a repository.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(gh api *)
  - Bash(gh issue list *)
  - Bash(gh pr list *)
  - Bash(gh search *)
  - Bash(git *)
  - Bash(curl *)
  - Bash(find *)
  - Bash(docker *)
  - Bash(/tmp/pcval/publiccode-parser-go *)
  - Bash(npx @stoplight/spectral-cli *)
  - WebFetch(*)
---

# Genereer een publiccode.yml

Maak een `publiccode.yml` aan in de root van het huidige project op basis van de
informatie in de repository.

## Instructies

- Gebruik de documentatie op https://yml.publiccode.tools/schema.core.html om de
  keys te interpreteren.
- Als het project al lokaal een publiccode.yml heeft, gebruik dan die en
  vergelijk deze met de nieuwe informatie.

## Projectinformatie verzamelen

Zoek naar de volgende bestanden en lees ze om projectinformatie te verzamelen:

- `README.md` — naam, beschrijving, doel van het project
- Doorzoek het project op een `openapi.json` bestand. Gebruik de inhoud hiervan
  bij het genereren van de publiccode.yml.
- `package.json` / `pom.xml` / `pyproject.toml` / `Cargo.toml` / `go.mod` —
  controleer of er nuttige eigenschappen in staan.
- Bestaande lokale `publiccode.yml` — als die bestaat, werk deze bij in plaats
  van een nieuwe aan te maken.
- `.git/config` of remote URL — voor de `url` van de repository.

Haal aanvullende informatie op via de remote:

- Gebruik de repository-URL om projectinformatie op te halen van de git-omgeving
  (GitHub / GitLab / Forgejo).
- Haal voor `maintenance.contacts[].name` de weergavenaam van de
  GitHub-organisatie op via de organisatiepagina (bijv.
  `https://github.com/<org>`). Gebruik de weergavenaam zoals getoond op de
  profielpagina, niet de slug uit de URL.
- Gebruik diezelfde weergavenaam ook als `legal.mainCopyrightOwner`.

## landingURL bepalen

Zoek naar een live URL waarop de software draait (bijv. in README, `one.json`,
CI/CD-configuratie of andere configuratiebestanden). Als die gevonden wordt,
voeg deze toe als `landingURL` direct onder `url`. Zet de URL **niet** in de
`longDescription`.


## Stap — Licentie bepalen

Bekijk of er een licentie in de repo staat, bepaal de SPDX-identifier en gebruik
die als `legal.license`.

Als er **geen** LICENSE-bestand aanwezig is in de root van het project, maak dan
een `LICENSE` bestand aan met de volledige Nederlandse tekst van de EUPL-1.2
licentie. De contents van de license ophalen op:
https://interoperable-europe.ec.europa.eu/sites/default/files/inline-files/EUPL%20v1_2%20NL.txt

Gebruik daarna `EUPL-1.2` als waarde voor `legal.license` in de publiccode.yml.

## Stap — softwareType bepalen

Gebruik de volgende criteria (in volgorde van prioriteit):

- Bevat het project een plugin-manifest (`plugin.json`, `.claude-plugin/`,
  `.vscode/`, etc.) of is het expliciet een extensie/addon voor een andere
  applicatie? → `addon`
- Zijn het alleen configuratiebestanden zonder uitvoerbare code? →
  `configurationFiles`
- Is het een herbruikbare bibliotheek zonder eigen UI of entrypoint? → `library`
- Heeft het een webinterface? → `standalone/web`
- Draait het als achtergrondservice of API zonder eigen UI? →
  `standalone/backend`
- Heeft het een desktopinterface? → `standalone/desktop`
- Anders → `standalone/other`

## Stap — Ontbrekende informatie vaststellen

Bepaal welke verplichte velden na de vorige stappen nog ontbreken. Stel de
gebruiker gerichte vragen voor elk ontbrekend veld.

### Naam

Zorg dat de sleutel `name` een normale titel bevat, geen kebab-case, camelCase,
PascalCase of snake_case.

### releaseDate en softwareVersion

`releaseDate` en `softwareVersion` sleutels **niet** toevoegen. Als ze bestaan,
verwijder ze dan.

### Lege regels na blokken met sub-sleutels

Na een sleutel met meerdere sub-sleutels moet altijd een lege regel worden
toegevoegd. Enkelvoudige sleutels (zoals `name`, `url`, `softwareType`,
`developmentStatus`) krijgen géén lege regel erna.

Correct:

```yaml
name: Mijn Project
url: https://example.com
softwareType: addon
developmentStatus: development

description:
  nl:
    shortDescription: ...

legal:
  license: EUPL-1.2
```

### longDescription en shortDescription opmaak

Schrijf `longDescription` als een YAML literal block scalar (`|`) met harde
regelafbrekingen zodat elke regel maximaal 80 tekens breed is. Voorbeeld:

```yaml
longDescription: |
  De OAS Generator is een webapplicatie waarmee ontwikkelaars eenvoudig een
  OpenAPI Specification (OAS) document kunnen genereren dat voldoet aan de
  Nederlandse API Design Rules.
```

Gebruik **geen** folded block scalar (`>`) voor `longDescription`, want die
voegt regels samen tot één lange regel.

Schrijf `shortDescription` als een YAML folded block scalar met strip-chomping
(`>-`) zodat elke regel maximaal 80 tekens breed is maar de waarde één
aaneengesloten string blijft (regelafbrekingen worden samengevoegd tot spaties).
Voorbeeld:

```yaml
shortDescription: >-
  Deze generator kan worden gebruikt om een OpenAPI-specificatie te maken dat
  voldoet aan de NL API Design Rules.
```

## Valideer met publiccode-parser-go

Valideer de publiccode.yml met https://github.com/italia/publiccode-parser-go.

## Referenties

- [publiccode.yml schemadocumentatie](https://yml.publiccode.tools/schema.core.html)
- [Categorielijst](https://yml.publiccode.tools/categories-list.html)
- [SPDX licentie-identifiers](https://spdx.org/licenses/)
