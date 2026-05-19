---
name: generate-publiccode-yml
description: |
Generate or update a publiccode.yml file for open source projects. 
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

# Generate a publiccode.yml

Create a file `publiccode.yml` in the root of the current project, using
the information available in the repository.

## Instructions

- Use the documentation at https://yml.publiccode.tools/schema.core.html
  to interpret keys.
- If the project already has a local publiccode.yml, use that and
  compare it with the new information.
  
## Gather information about the project

Search for the following files and read them to collect project information:

- Check `README.md` — for the name, description, and purpose of the project
- Search the project to see if it has an `openapi.json` file. Use its
  content to generate the publiccode.yml.
- Check for `package.json` / `pom.xml` / `pyproject.toml` / `Cargo.toml`
  / `go.mod` — check whether it contains any useful properties.
- If a local `publiccode.yml` exists, update that instead of creating a
  new one.
- Check `.git/config` or the remote URL for the repository `url`.

Get additional information via the remote:

- Use the repository URL from the Git environment to retrieve project
  information (GitHub / GitLab / Forgejo).
- For `maintenance.contacts[].name`, check for an organization name
  displayed on GitHub's "organization page" `https://github.com/<org>`,
  for example. Use the display name as shown on the profile page, not
  the slug from the URL.
- Use the same display name as `legal.mainCopyrightOwner`.

## Determine the LandingURL 

Search for a live URL where the software is hosted (e.g. in README,
`one.json`, CI/CD configuration, or other configuration files). When
available, add it as `landingURL` directly under `url`. Do **not** use
this URL for the `longDescription`.

## Determine the Licence 

Check if there is a license in the repo, determine the SPDX identifier
and use it for `legal.license`.

If there is **no** LICENSE file in the root of the project, create a
`LICENSE` file containing the full English text of the EUPL-1.2 license.
Retrieve the contents of the license here:
https://joinup.ec.europa.eu/sites/default/files/custom-page/attachment/2020-03/EUPL-1.2%20EN.txt

Use `EUPL-1.2` as the value for `legal.license` in the publiccode.yml file.

## Determine the SoftwareType 

Use the following criteria, in this order of priority:

- Does the project include a plugin manifest (`plugin.json`, `.claude-plugin/`,
  `.vscode/`, etc.) or is it explicitly an extension/addon for another
  application? → `addon`
- Does the project consists solely of configuration files with no
  executable code? → `configurationFiles`
- Is it a reusable library without a UI or entry point? → `library`
- Does it have a web interface? → `standalone/web`
- Does it run as a background service or API without own UI? →
  `standalone/backend`
- Does it have a desktop interface? → `standalone/desktop`
- Other → `standalone/other`


## Name

Make sure the key `name` is a normal written title, and not in either
kebab case, camelCase, PascalCase or snake_case.

## releaseDate and softwareVersion

Do **not** add `releaseDate` and `softwareVersion`. If these exist,
remove them.

## Newlines delimit blocks with sub-keys

Add a newline following a key with multiple sub-keys. Do not add a
newline after single keys such as `name`, `url`, `softwareType`,
`developmentStatus`.

Correct:

```yaml
name: My Project
url: https://example.com
softwareType: addon
developmentStatus: development

description:
  en:
    shortDescription: ...

legal:
  license: EUPL-1.2
```

## Print longDescription en shortDescription

Print `longDescription` as a YAML literal block scalar (`|`) with hard
line breaks preserved, so that each line is a maximum of 80 characters
wide. Example:

```yaml
longDescription: |
  The OAS Generator is a web application allowing developers to easily
  generate a OpenAPI Specification (OAS) document that complies with the
  Dutch API Design Rules.
```


Do **not** use folded block scalar (`>`) for `longDescription`, as that
will cause lines to be merged.

Print `shortDescription` as a YAML folded block scalar with
strip-chomping (`>-`) so that each line is a maximum of 80 characters
wide, while keeping the value a single contiguous string (line
terminations are merged into spaces). Example:

```yaml
shortDescription: >-
  This generator can be used to create an OpenAPI specification to
  comply with the NL API Design Rules.
```

## Validate using publiccode-parser-go

Validate publiccode.yml using https://github.com/italia/publiccode-parser-go.

## References

- [publiccode.yml schema documentation](https://yml.publiccode.tools/schema.core.html)
- [List of categories](https://yml.publiccode.tools/categories-list.html)
- [SPDX licence identifiers](https://spdx.org/licenses/)
