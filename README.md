# moi

**moi** (French for *"me"*) is a convention for organizing your own personal files


**directly over a directory you already own** — by default your `~/Documents`. Your real files *are*
the moi data. Its purpose is to be a durable, plain-text **personal-context substrate**: a place any
tool or AI agent can read to learn who you are and what's going on in your life. 

```
~/Documents/            ← $MOI_HOME (your real Documents folder)
├── bio.txt              # who you are + how agents should help you
├── todo.txt            # active tasks   (todo.txt format)
├── done.txt            # completed tasks
├── tasks/              # one plan .md per in-progress task (done/ archives finished ones)
│   ├── finish-the-novel.md
│   └── done/           #   completed plans, by YYYY/MM/DD
├── project.yaml        # optional: marks the root as the "home" project
├── notes/              # free-form personal knowledge base (plain markdown)
│   ├── health.md
│   └── finances.md
├── journal/            # dated log entries (YYYY-MM-DD.md)
│   └── 2026-05-26.md
├── avatar.txt          # this is you — open key-value doc, no schema
├── intake.log.jsonl    # append-only dose/use event log — schema'd JSONL
├── drives/             # storage catalog — one file per drive (named by label)
│   └── vault.yaml
├── email/              # saved .eml messages, by account + date
│   └── you@example.com/2026/05/28/HH-MM-SS - sender - subject.eml
├── contacts/           # address book — one .vcf (vCard) per contact
│   └── Sam Doe.vcf
├── private/            # most-sensitive stage — secure it (perms / no-sync / encrypt)
│   └── avatar.txt      #   address, health, substances…
│
├── projects/           # your project workspaces
│   └── novel/          #   a project — contains a project.yaml
│       ├── project.yaml
│       ├── chapter-01.md   # ...alongside your actual files
│       └── outline.md
├── scripts/            # your helpers; scripts/converters/ for §10 converters
│   └── converters/
├── taxes-2026/         # just your files — moi leaves it alone
└── recipes/
```

## The whole idea in five rules

1. **The root is a folder you own.** `$MOI_HOME` defaults to `~/Documents`. moi reserves only a few
   names there — `bio.txt`, `avatar.txt`, `todo.txt`, `done.txt`, `project.yaml`,
   `intake.log.jsonl`, `tasks/`, `notes/`, `journal/`, `drives/`, `email/`, `contacts/`, `projects/`, `scripts/`, `private/`, `secret/` — and ignores everything else.
2. **Personal context is the point.** `bio.txt` (how to help you), `avatar.txt` (an open key-value
   "this is me"), `notes/` (knowledge base), and `journal/` (dated log) are plain text; the only
   schema-validated personal file is `intake.log.jsonl` (a dose/use log). An agent reads these to gain
   standing context instead of starting from zero. All optional.
3. **Projects live in `projects/`.** Each is a folder under `projects/` with a `project.yaml` (carrying
   its id/name/type and any `x-<tool>:` state). Anything outside `projects/` is just your files.
4. **Tools share the tree via namespaces.** Each tool keeps its own state under an `x-<tool>:` key in
   `project.yaml` and must preserve the others on write. So a CLI, a dashboard, and an agent can all
   operate on the same files at once.
5. **Privacy comes in stages.** Keep everything in the base, or move the most sensitive parts into a
   more-protected stage directory — `private/` (or `secret/`) — that mirrors the root and that you
   secure however you like (permissions, no-sync, encryption). A tool merges the stages it can read.

## Why

Your todos, notes, and projects outlive any single app. A plain-text, plain-YAML overlay means no
lock-in (edit by hand, by `grep`, by any editor), multiple concurrent consumers, and no runtime
dependency that can be switched off.

## Read this

- **[`SPEC.md`](SPEC.md)** — the authoritative convention: root, personal context (`bio.txt`,
  `avatar.txt`, `notes/`, `journal/`), task files, the project marker rule, the `project.yaml` core
  schema, and the `x-<tool>:` extension mechanism.
- **[`schema/`](schema/)** — JSON Schemas (Draft 2020-12) for the structured files: `project.yaml`,
  one `intake.log.jsonl` entry, and one `drives/` file.
- **[`examples/documents/`](examples/documents/)** — a complete reference layout you can read.

## Validate a layout

Any JSON Schema validator works (no moi-specific tooling required). For example, with
[`check-jsonschema`](https://github.com/python-jsonschema/check-jsonschema):

```sh
# project manifests: the root home project, plus every project one level down
check-jsonschema --schemafile schema/project.schema.json \
  ~/Documents/project.yaml ~/Documents/*/project.yaml

# the JSONL intake log — validate each line (check-jsonschema reads stdin via '-')
while IFS= read -r line; do
  printf '%s' "$line" | check-jsonschema --schemafile schema/intake-entry.schema.json -
done < ~/Documents/intake.log.jsonl

# the drives catalog — one file per drive
check-jsonschema --schemafile schema/drive.schema.json ~/Documents/drives/*.yaml
```
