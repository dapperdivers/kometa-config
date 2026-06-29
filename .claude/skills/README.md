# Kometa authoring skills

[Agent Skills](https://agentskills.io) for writing this Kometa configuration.
Each skill follows the same discipline: **read the live Kometa source before
proposing YAML**, because this config tracks the **nightly** branch where syntax
moves fast.

| Skill                                               | Owns                                                                           | Read first                                                             |
| --------------------------------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| [`kometa-collections`](kometa-collections/SKILL.md) | collection / dynamic-collection / metadata files, builders, filters, defaults  | `kometa.wiki/en/nightly/files/...` + `defaults/` source                |
| [`kometa-overlays`](kometa-overlays/SKILL.md)       | overlay files, positioning, group/weight/queue, overlay defaults, images/fonts | `kometa.wiki/en/nightly/files/overlays/` + `defaults/overlays/` source |
| [`kometa-config`](kometa-config/SKILL.md)           | `config.yml` libraries, settings, operations, playlists, integrations          | `kometa.wiki/en/nightly/config/...`                                    |
| [`kometa-scheduling`](kometa-scheduling/SKILL.md)   | `schedule` / `visible_home` / `visible_shared` values and rotations            | `kometa.wiki/en/nightly/config/schedule/`                              |

## Shared conventions these skills assume

- **Branch:** nightly. Docs `https://kometa.wiki/en/nightly/<path>/`, raw
  `https://raw.githubusercontent.com/Kometa-Team/Kometa/nightly/<path>`.
- **List source paths** with
  `gh api "repos/Kometa-Team/Kometa/git/trees/nightly?recursive=1" --jq '.tree[].path | select(...)'`.
- **In-container paths:** files referenced as `/config/...` (repo root mounts at
  `/config`).
- **Validate:** `mise run validate` (offline Kometa check of config.yml + all linked
  files via the pinned `kometateam/kometa:nightly` Docker image — no local install) or
  `mise run kometa -- --validate-file /config/<file>.yml --validate-level structure` for one
  file. `oxfmt` (`.oxfmtrc.json`) handles formatting via the lefthook pre-commit hook;
  never edit `*_report.yml` or `logs/` (generated/excluded).
- **Live test:** `testing/` Docker harness (`testing/docker-compose.yml`,
  `testing/test-overlays.sh`) runs against a throwaway Plex — never test against
  production.
- **Secrets:** integrations use `<<PLACEHOLDER>>` variables; never hardcode keys.

## Editing a skill

Keep each `SKILL.md` under ~500 lines and push deep catalogs into `references/`.
After changes, re-validate frontmatter: `name` must equal the folder name,
lowercase/numbers/hyphens only, and `description` ≤ 1024 chars.
