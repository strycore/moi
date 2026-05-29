# The moi standard

moi is a convention for organizing your personal files over a directory by default `~/Documents` 
A moi tool is any program that reads or writes a moi tree.

---

## 1. The root

moi data lives under a directory referred to as `$MOI_HOME`. The default location is `~/Documents`

```
~/Documents/            ← $MOI_HOME (your real Documents folder)
├── avatar.txt           ← this is you (free-form)              (§3.4)
├── todo.txt            ← active tasks                                (§2)
├── done.txt            ← completed tasks                             (§2)
├── tasks/              ← in-progress task plans                      (§2.1)
│   ├── finish-the-novel.md
│   └── done/           ←   completed plans, archived by YYYY/MM/DD
├── notes/              ← free-form personal knowledge base           (§3)
│   ├── health.md
│   └── finances.md
├── journal/            ← dated log entries                           (§3)
│   └── 2026-05-26-some-title.md
├── intake.log.jsonl    ← append-only dose/use event log              (§3.5)
├── drives/             ← catalog of your storage, one file per drive  (§3.8)
│   └── vault.yaml      ←   named by label; uuid lives inside
├── email/              ← saved .eml messages, by account + date       (§3.9)
│   └── you@example.com/2026/05/28/HH-MM-SS - sender - subject.eml
├── contacts/           ← address book, one .vcf per contact          (§3.10)
│   └── Sam Doe.vcf
├── records/            ← scanned docs & credentials, by category     (§3.11)
│   ├── health/2021-12-31 - covid-19 vaccination card (CA, Moderna x3).png
│   └── receipts/
│       ├── recipients.txt
│       └── 2026-05-28 - kaiser - receipt 3952863.pdf
├── secret/             ← encrypted-at-rest stage: credentials,        (§3.7)
│   │                       tokens, SSN/passport/CC, sensitive files
│   └── …                   (layout is the user's; secured by user)
│
├── projects/           ← your project workspaces                     (§4)
│   └── novel/          ←   a project (contains a project.yaml)
│       ├── project.yaml
│       ├── chapter-01.md
│       └── outline.md
├── scripts/            ← your helpers; scripts/converters/ for §10    (§11)
│   └── converters/
├── taxes-2026/         ← just your files — moi leaves it alone
└── recipes/
```

The names moi reserves at the root are `bio.txt`, `avatar.txt`, `todo.txt`, `done.txt`, `project.yaml`,
`intake.log.jsonl`, and the `tasks/`, `notes/`, `journal/`, `drives/`, `email/`, `contacts/`, `records/`,
`projects/`, `scripts/`, and `secret/` directories (the last is the encryption stage, §3.7).
Everything else under `$MOI_HOME` belongs to the user.

These names are matched **case-insensitively**: a reader **MUST** treat `Records/`, `records/`, and
`RECORDS/` as the same reserved entry, and **MUST** preserve whatever casing the user chose rather than
renaming it (capitalized forms like `Records/` or `Journal/` — as in the XDG user dirs `Music`,
`Pictures` — are equally valid). A tree **MUST NOT** contain two reserved entries that differ only in
case (§7).

---

## 2. Tasks — `todo.txt`, `done.txt`, `tasks/`

- `$MOI_HOME/todo.txt` holds active tasks; `$MOI_HOME/done.txt` holds completed tasks. They are the
  task lists of the root **home** project (§4.3).
- Both files **MUST** follow the [**todo.txt format**](https://github.com/todotxt/todo.txt/blob/master/description.md).
  That specification is **authoritative** for all formatting details — priorities, completion marks,
  creation/completion dates, `+project` and `@context` tags, and `key:value` metadata. moi adds no
  rules of its own; it relies on todo.txt being widely understood by tools and agents. (Brief reminder:
  one task per line; `(A)` priority; a completed line begins with `x` followed by the completion date.)
- A task **MAY** associate itself with a moi project using a `+<slug>` tag matching that project's
  `slug` (§4.1).
- Both files **MAY** be absent; a tool treats an absent file as empty.

### 2.1 In-progress plans — `tasks/`

`$MOI_HOME/tasks/` holds a working document for each task you have actively started — one free-form
markdown file per task (e.g. `tasks/finish-the-novel.md`), holding its goal, plan, and progress. Where
`todo.txt` is the one-line list, a `tasks/` file is the expanded plan; the folder's contents are, in
effect, your in-progress set. A plan file **SHOULD** be named after the `+<slug>` tag on its `todo.txt` line, linking the list entry
to its plan.

A task moves through this lifecycle:

1. **Capture** — add a line to `todo.txt` (§2), tagged `+<slug>`.
2. **Plan & validate** — create `tasks/<slug>.md` with the goal and steps, and confirm it holds
   everything needed to execute: URLs, credentials, contacts, file paths, etc. (Sensitive inputs such
   as credentials **SHOULD** live in a privacy stage, §3.7.)
3. **Execute** — do the work, updating the plan as you go.
4. **Complete** — mark the `todo.txt` line done and move it to `done.txt` (§2).
5. **Archive** — move the plan to `tasks/done/<YYYY>/<MM>/<DD>/<slug>.md`, dated by completion.

So the files directly under `tasks/` are exactly your active set; `tasks/done/` is the dated archive of
finished plans.

**Linking to issue trackers.** A task **MAY** be associated with external GitHub issues or Jira tickets
via `key:value` metadata on its `todo.txt` line (todo.txt's native syntax, §2) — recommended
`gh:<owner>/<repo>#<n>` for GitHub and `jira:<KEY>-<n>` for Jira, repeated for several. The plan
(`tasks/<slug>.md`) **MAY** also list the full URLs and cross-references. Keeping the reference on the
task lets a tool sync status between moi and the tracker; such syncing is a tool's job (§9), not part of
the core.

---

## 3. Personal context (the home project)

This is moi's reason to exist: a record of who you are and what's happening in your life, so an agent
has standing context instead of starting from zero each time. It all lives in the home project (the
root, §4.3) and every file is **OPTIONAL** — an absent file or directory simply means "none".

Two kinds of content live here:

- **Plain text** — `bio.txt`, `notes/`, and `journal/` are prose; `avatar.txt` is an open key-value
  document. No schema, no required frontmatter, no index (§3.1–§3.4).
- **Structured records** — the dose & substance log (§3.5) and the drives catalog (§3.8) are
  schema-validated, because they must be read identically by every tool.
- **Standard-format files** — `email/` keeps messages as `.eml` (§3.9) and `contacts/` keeps your
  address book as `.vcf` vCards (§3.10).
- **Documents** — `records/` keeps scanned documents & credentials in category subfolders (health,
  bank, rent, receipts, …) (§3.11).

> **Sensitivity.** The whole tree is personal by default — it lives on a device the user controls. moi
> says nothing about access control; protecting the tree (filesystem permissions, an encrypted home,
> selective sync) is the user's responsibility and out of scope. For the subset that warrants
> encryption at rest — credentials, sensitive identifiers, anything too personal to leave in cleartext —
> moi reserves `secret/` (§3.7). A tool **MUST NOT** transmit any of this content anywhere without
> explicit user intent.

### 3.1 `bio.txt` — the entry point

`$MOI_HOME/bio.txt` is the canonical "read me first" document. An agent building context **SHOULD**
read it before anything else. It describes:

- **Identity** — who the user is: background, situation, the people and things that matter.
- **Standing guidance** — how the user wants agents to behave: preferences, tone, boundaries, and any
  always-applicable instructions (a personal, global equivalent of a project's agent instructions).

It is a single human-written plain-text file. A tool **SHOULD NOT** rewrite it wholesale; updates
**SHOULD** be small, targeted edits (§7).

### 3.2 `notes/` — knowledge base

`$MOI_HOME/notes/` holds free-form markdown notes, one topic per file by convention
(e.g. `notes/health.md`, `notes/finances.md`). Notes **MAY** reference each other with ordinary
relative markdown links. No structure is imposed; a note is just a markdown file. Tools **MAY** create,
read, and edit notes, and **MUST** preserve content they did not author (§7).

### 3.3 `journal/` — dated log

`$MOI_HOME/journal/` holds dated entries, one file per day named `YYYY-MM-DD.md` (ISO 8601, e.g.
`journal/2026-05-26.md`). Entries are an append-oriented record of what's going on over time. A tool
appending to "today" **SHOULD** create the file if absent and add to it rather than overwrite earlier
content.

### 3.4 `avatar.txt` — this is you

`$MOI_HOME/avatar.txt` is the canonical "this is me" file an agent reads to know who it's dealing with.
It is a **key-value document** — one `key: value` per line — not prose. The keys are **open**: they can
name any aspect of your life (name, pronouns, body, health, interests, work, devices, …), and moi
imposes no schema on which keys exist or what their values mean.

Some keys hold **sub-entries** instead of a single value — an indented block under the key, of further
`key: value` lines or a list. For example `projects:`, `emails:`, `emergency contacts:`, and `machines:`
each group several items. See [`examples/documents/avatar.txt`](examples/documents/avatar.txt) for the
shape.

It is yours to shape: a tool **MUST NOT** reformat it or impose a schema, and **MUST NOT** rewrite or
delete your entries. It may append, or make small targeted edits, only at your request (§7).

### 3.5 Dose & substance log — `intake.log.jsonl`

What you take on an ongoing basis — regimen, medicines, supplements — is recorded free-form in the
avatar (§3.4). The one structured file here is the **event log**: each dose, use, or purchase, timestamped.

`$MOI_HOME/intake.log.jsonl` is an append-only log in [JSON Lines](https://jsonlines.org/) (one JSON
object per line) recording each substance event. `at` (an RFC 3339 / ISO 8601 timestamp *with* timezone
offset, e.g. `2026-05-28T09:12:00+02:00`) and `substance` are required; `action` (`taken` or
`purchased`, default `taken`), `dose`, `category` (`medical`, `recreational`, `supplement`, or
`other`), `route`, `reason`, `notes` are optional. Each line conforms
to [`schema/intake-entry.schema.json`](schema/intake-entry.schema.json). A writer **MUST** append new
lines and **MUST NOT** rewrite earlier ones (line-append is also the safest concurrent write; §7).

### 3.6 Building context (for agents)

To assemble personal context, an agent **SHOULD** start at `bio.txt`, then consult the avatar (§3.4)
for hard facts and `notes/`, recent `journal/` entries, and the dose log (§3.5) as the task warrants,
plus the `home` project's tasks (§2), and merging in anything it can read from `secret/` (§3.7). It
**MUST** treat all of this as the user's own data: cite or summarize it, but never silently rewrite or
delete it, and never copy content out of `secret/` into the base tree.

### 3.7 The `secret/` stage

The whole moi tree already lives on a device the user controls, so it is **personal by default** — there
is no separate "private" stage and no "public" tag. The user decides what (if anything) to share from
this tree on a case-by-case basis.

The one reserved exception is `$MOI_HOME/secret/`, a stage for material that warrants encryption at rest
even on this machine — credentials, tokens, sensitive identifiers (SSN, passport, CC), and any files or
directories the user judges too sensitive to leave in cleartext. Its layout is up to the user (e.g. one
keyring file per credential); moi only fixes the location.

moi says **nothing about how `secret/` is secured** — filesystem permissions, exclusion from sync, an
encrypted volume or external drive, or an `age`/`gpg` blob (e.g. `secret.tar.age`) kept in the
synced tree and decrypted only when used. Whichever mechanism is chosen, the key or passphrase **MUST**
live *outside* the moi tree — never synced beside the blob — and losing it means losing the stage.

A tool **MUST NOT** assume `secret/` is readable (no permission, undecryptable, absent on this machine
all mean the same thing: less context), and **MUST NOT** copy content from `secret/` into the base tree.

### 3.8 Drives catalog — `drives/`

`$MOI_HOME/drives/` is a structured inventory of your storage media — one YAML file per filesystem,
named by a **friendly name** (the drive's `label` if it has one, otherwise one you choose, e.g.
`drives/vault.yaml`). It records what drives you have and where: the kind of thing a file-indexer
produces and a backup tool or agent consults.

Each file carries at least `uuid`; commonly also `label`, `fs`, `size`, `model`, `serial`, `last_mount`,
`device` (the machine it's attached to), `privacy` (`public` / `personal` / `confidential` — the same
gradient as the stages of §3.7), and free `notes`. Validated by
[`schema/drive.schema.json`](schema/drive.schema.json); a tool **MAY** add its own keys (or an
`x-<tool>:` namespace, §5) and **MUST** preserve ones it doesn't own (§7). The `uuid` field — not the
filename — is the authoritative key, so the same external disk reconciles to a single entry across
machines (a tool **MAY** rename the file if the label changes). A scanner populates these files; a
human annotates `label`/`privacy`/`notes`. See §9 for the fili integration.

A `folders:` map catalogs the drive's top-level folders, keyed by name. Each value is empty (just
listed) or carries per-folder metadata: `privacy` (`public`/`personal`/`confidential`), `backup_of`
(what the folder backs up — a drive label, path, device, or project; its presence marks the folder a
backup), and `notes`.

### 3.9 Email archive — `email/`

`$MOI_HOME/email/` archives saved messages as standard `.eml` files (RFC 822 / MIME — the portable
on-disk email format any client can read), laid out by account then date:

```
email/<account-address>/<YYYY>/<MM>/<DD>/HH-MM-SS - <sender> - <subject>.eml
```

- `<account-address>` is the mailbox the message belongs to — one subtree per account you archive
  (e.g. `you@gmail.com/`, `you@work.example/`).
- `YYYY/MM/DD` and the `HH-MM-SS` filename prefix come from the message's own date.
- The filename is `time - sender - subject`, with all parts filesystem-safe (§7): the time written as
  `HH-MM-SS`, and disallowed characters in the sender/subject replaced.
- Mail is sensitive: the `email/` tree **SHOULD** usually live in a privacy stage (§3.7), not the
  public base.

### 3.10 Contacts — `contacts/`

`$MOI_HOME/contacts/` is your address book — one [vCard](https://datatracker.ietf.org/doc/html/rfc6350)
file per contact (`.vcf`, vCard 4.0 recommended), the portable standard any phone or mail client reads.
Name each file by the contact's display name, filesystem-safe (§7) — e.g. `contacts/Sam Doe.vcf`; the
vCard's `UID` is the stable identity inside, so the filename can change without losing it. Contacts hold
other people's personal data and **SHOULD** usually live in a privacy stage (§3.7). (The avatar's
`emergency contacts:` is a quick shortlist; the full records live here.)

### 3.11 Records — `records/`

`$MOI_HOME/records/` holds scanned official documents and credentials — IDs, passports, insurance and
membership cards, certificates, licenses, vaccination and other health records, receipts, and the like
(images or PDFs) — organized into **category subfolders**. The category set is open; add what you need:

```
records/health/    records/bank/    records/rent/    records/receipts/    …
```

Within a category, name each file `<YYYY-MM-DD> - <description>.<ext>` (filesystem-safe, §7), the date
being when the record was issued or last effective — e.g.
`records/health/2021-12-31 - covid-19 vaccination card (CA, Moderna x3).png`. In a multi-person home you
**MAY** prefix the person.

**Receipts** (`records/receipts/`) are a category with one extra rule: the filename also carries a known
**recipient identifier** — the payee — drawn from `records/receipts/recipients.txt` (one `id: Full Name`
per line, e.g. `kaiser: Kaiser Permanente`), so a payee is always named the same way:
`<YYYY-MM-DD> - <recipient> - <description>.<ext>`, e.g. `2026-05-28 - kaiser - receipt 3952863.pdf`.

Records are sensitive and **SHOULD** usually live in a privacy stage (§3.7).

---

## 4. Projects

> **Projects live under `$MOI_HOME/projects/`; each is a directory there that contains a `project.yaml`.**

Scoping projects to a single `projects/` folder keeps moi from having to guess about arbitrary
directories — anything outside `projects/` is simply the user's own files.

- A tool **SHOULD** discover projects by scanning the immediate children of `$MOI_HOME/projects/` for a
  `project.yaml`. It **MAY** also track projects whose directories live elsewhere (e.g. a code repo
  under `~/Projects`); how such external paths are remembered is tool-specific and out of scope here.
- The project's identity comes from its `project.yaml` (§4.1), not from its directory name. A project
  directory **MAY** have any name and **MAY** contain the user's actual working files alongside the
  manifest.

### 4.1 `project.yaml` — core fields

`project.yaml` is a YAML map. The following **core fields** are owned by the moi standard:

| Field         | Type                | Req. | Notes |
|---------------|---------------------|------|-------|
| `id`          | string              | yes  | Stable, unique within the tree. A UUID is **RECOMMENDED**. The literal `home` is reserved (§4.3). Immutable. |
| `name`        | string              | yes  | Human-readable label. |
| `slug`        | string              | no   | Stable, filesystem-safe identifier (`^[a-z0-9][a-z0-9-]*$`) used for cross-references such as `+slug` task tags and tool URLs. It is **independent of the directory name** (real directories may be `My Novel`, `Photos`, …). If present it **SHOULD** be unique within the tree and **SHOULD NOT** change. |
| `type`        | enum                | no   | One of `software`, `research`, `personal`, `other`, `home`. Defaults to `other`. |
| `created_at`  | string (ISO 8601)   | no   | Creation timestamp. |
| `updated_at`  | string (ISO 8601)   | no   | Last-modified timestamp; writers **SHOULD** update it on every change. |
| `repos`       | list of strings     | no   | Absolute paths / identifiers of code repositories or directories this project spans. A UX hint. |

A tool **MUST** preserve core fields it does not use, and **MUST NOT** repurpose a core field name
for tool-specific data — use an extension namespace (§5) instead.

### 4.2 Round-trip preservation

`project.yaml` is the source of truth and is intended to be hand-editable. Writers **MUST** preserve,
on round-trip, every top-level key they do not own (§5), and **SHOULD** preserve user comments and key
ordering (e.g. by using a round-tripping YAML library such as `ruamel.yaml`; see §7).

### 4.3 The home project (the root)

The root directory `$MOI_HOME` is itself the **home** project: the pinned personal workspace for data
not tied to any single project. Its content is the personal context of §3 (`bio.txt`, `notes/`,
`journal/`) and its tasks (§2). The home is the root itself — not a directory under `projects/` — and
exists as a project when the root contains `$MOI_HOME/project.yaml`. That manifest uses the reserved
values `id: home`, `slug: home`,
`type: home`. There **MUST NOT** be more than one project with `type: home`. Tools **SHOULD NOT** allow
deleting it. The root `project.yaml` is **OPTIONAL**; a tool **MAY** create it on first run.

---

## 5. Tool extensions — `x-<tool>:` namespaces

Tools store their own state in `project.yaml` under a **namespaced top-level key** matching:

```
^x-[a-z0-9][a-z0-9-]*$
```

The portion after `x-` is the tool's name. A tool:

- **MUST** write its state only under its own `x-<tool>:` key (the value **MAY** be any YAML).
- **MUST NOT** read, modify, or delete another tool's `x-<...>:` namespace.
- **MUST** preserve unknown `x-<...>:` namespaces on round-trip write (see §4.2).

This is what lets several tools share one tree (see the illustrative example in §6).

A `project.yaml` **MUST NOT** contain top-level keys that are neither core fields (§4.1) nor `x-*`
extension namespaces.

---

## 6. Extension example (non-normative)

moi assigns no meaning to any `x-<tool>:` namespace — its contents are entirely the owning tool's
concern. The following is purely illustrative: a hypothetical editor named `acme` that remembers which
file was last open and how its panes were arranged.

```yaml
x-acme:
  last_opened: chapter-01.md
  panes:
    - file: chapter-01.md
      cursor: { line: 42, col: 7 }
    - file: outline.md
```

Another tool reading this manifest **MUST** leave the `x-acme:` block exactly as it found it.

---

## 7. Notes for tools and agents

Guidance for writing to a moi tree correctly. The preservation requirements (§3.6, §4.2) are
normative; the techniques below are the **RECOMMENDED** ways to satisfy them.

- **Don't blindly reserialize.** Reading a YAML file into a plain map and dumping it back loses
  comments and key order and so violates §4.2. This applies to the YAML moi defines —
  `project.yaml`. To edit safely, either (a) use a round-tripping
  parser (e.g. `ruamel.yaml`), or (b) — especially for an agent with only generic file tools — make a
  **minimal targeted text edit** to the specific line(s) you are changing and leave the rest of the
  bytes untouched. The same care applies to the user's prose in `bio.txt`, `avatar.txt`, and `notes/`.
  (`intake.log.jsonl` is append-only and so sidesteps this entirely — add a line, never rewrite the file.)
- **Write atomically.** Several tools may share the tree concurrently. A writer **SHOULD** write to a
  temporary file in the *same directory* and `rename()` it over the target, so a reader never observes
  a half-written file. This applies to manifests, task files, and context documents alike.
- **Use filesystem-safe names.** Every file and directory name moi creates **MUST** be portable across
  the filesystems in common use today (ext4, btrfs, XFS, APFS, NTFS, exFAT, FAT32, …). Concretely: never
  use the characters `< > : " / \ | ? *` or control characters; don't end a name with a space or a dot;
  avoid the Windows-reserved names (`CON`, `PRN`, `AUX`, `NUL`, `COM1`–`COM9`, `LPT1`–`LPT9`); keep each
  path component ≤ 255 bytes; and don't rely on letter-case or Unicode-normalization differences to tell
  two names apart (some filesystems fold them together). A writer **MUST** sanitize derived names —
  timestamps in a colon-free form (`HH-MM-SS`), and disallowed characters from titles, subjects, or
  senders replaced (e.g. with `-`).
- **Validate before writing.** A writer **SHOULD** validate a changed `project.yaml` against
  `schema/project.schema.json` (any JSON Schema validator handles YAML) before committing it. This
  catches a stray top-level key or a malformed core field, and lets an agent self-check its own output.
- **Generate `id` once.** Set `id` when the manifest is created (a UUID is fine) and never change it;
  treat `slug` the same way.

---

## 8. Conformance checklist

A conforming moi tool:

1. Resolves `$MOI_HOME` (default `~/Documents`); treats it as a directory of the user's own files.
2. Claims only the reserved root names (`bio.txt`, `avatar.txt`, `todo.txt`, `done.txt`, `project.yaml`,
   `intake.log.jsonl`, `tasks/`, `notes/`, `journal/`, `drives/`, `email/`, `contacts/`, `records/`, `projects/`, `scripts/`, `secret/`); matches them case-insensitively (§1); leaves all other root entries untouched.
3. Reads/writes `todo.txt` / `done.txt` in todo.txt format, when it touches tasks (§2).
4. Treats `bio.txt`, `avatar.txt`, `notes/`, and `journal/` as free-form plain text (no schema), and
   `intake.log.jsonl` as a schema-validated structured log; never silently rewrites or deletes the
   user's data (§3).
5. Treats an inaccessible `secret/` stage as absent, and never copies content out of it into the
   base tree (§3.7).
6. Discovers projects by scanning `$MOI_HOME/projects/` for directories containing a `project.yaml`
   (plus the root's home manifest, and any external project paths it chooses to track) (§4).
7. Honors all core fields (§4.1), never repurposes their names, and treats `slug` as independent of the
   directory name.
8. Confines its own state to a single `x-<tool>:` namespace (§5).
9. Preserves unknown keys, namespaces, comments, and ordering on write (§4.2, §7).
10. Writes files atomically (temp file + rename) so concurrent consumers never see a partial write (§7).
11. Leaves unrecognized files and directories untouched.

---

## 9. Interoperability (non-normative)

This section is **informative**, not normative: moi neither requires nor depends on the tools below. It
documents how moi maps onto existing software — possible precisely because a moi tree is just plain files.

### Obsidian

[Obsidian](https://obsidian.md) is a markdown-vault editor, and a moi home is a compatible vault: open
`$MOI_HOME` as a vault and moi's markdown layer becomes first-class.

- `notes/` and `journal/` gain wikilinks, backlinks, graph view, and search. `notes/` is, in
  effect, what Obsidian is built for.
- `journal/` maps onto Obsidian's **Daily Notes** core plugin — point it at the `journal/` folder with
  date format `YYYY-MM-DD`. (A moi journal entry MAY carry a `-title` suffix; Daily Notes won't treat
  that as the canonical daily note, but the file is still an ordinary note.)
- The non-markdown files — `bio.txt`, `avatar.txt`, `todo.txt`/`done.txt`, `intake.log.jsonl`, `project.yaml`, and
  any encrypted stage blob — stay visible and editable but are **not** first-class Obsidian notes.
  Obsidian's frontmatter-based Properties/Bases features therefore don't apply, which is consistent with
  moi's personal files being intentionally free-form.
- Obsidian keeps its own config in `$MOI_HOME/.obsidian/` — an unrecognized directory moi tools ignore
  (§1). Exclude `.obsidian/workspace*` from sync to avoid conflicts.

A moi-aware Obsidian plugin **MAY** store its state under an `x-obsidian:` namespace in `project.yaml`
(§5), exactly as any other consumer.

### fili

[fili](https://github.com/strycore/fili) is a file-indexer that classifies everything on your drives.
It populates moi's `drives/` catalog (§3.8) — exporting one file per filesystem UUID — and reads it
back; its richer per-file index stays in its own store, with any moi-side extras under an `x-fili:`
namespace. fili's `public` / `personal` / `confidential` privacy levels mirror moi's stages (§3.7).

---

## 10. Application & service data (lifecycle)

Much of your data is trapped inside applications and online services — playlists in a music player,
history in a browser cache, posts in a Facebook archive, notes in Joplin or Tomboy, mail in
Thunderbird. moi treats any app setting or service capable of holding user data as worth liberating.
For each supported source, two things happen:

1. **Preserve the original.** The raw export or dump (a Facebook ZIP, a Joplin `.jex`, a Thunderbird
   profile, …) **SHOULD** be kept verbatim on a backup drive under a predictable location and a name
   that **includes the source data's last-modified timestamp** — recommended
   `<source>/<account-or-instance>/<last-modified>-<name>`, where `<last-modified>` is a filesystem-safe
   (§7), colon-free ISO 8601 stamp (e.g. `2026-05-28T143000`) of when the data itself last changed (not
   when you happened to export it). This makes each
   archived copy self-dating, its vintage obvious at a glance, and lets identical states de-duplicate.
   The original is the ground truth; conversion is lossy by nature.
2. **Convert into moi's plain-text conventions.** A per-source **extractor/converter** maps the export
   into the matching moi shape — e.g. Joplin or Tomboy notes → `notes/` (§3.2), Thunderbird mail →
   `email/` (§3.9), a Facebook archive → dated `journal/` entries or logs (§3.3). The converted form is
   what you read and search day to day; the preserved original lets you re-convert later as the
   converters improve.

Converters are per-source tools that live under `scripts/converters/` (§11), not part of the moi core
(like the consumers in §9); moi only fixes the destination conventions and the preserve-the-original
discipline. A converted artifact **SHOULD**
record its provenance (a `source:` line, or an `x-<tool>:` field) so it can be traced back to the
preserved original.

---

## 11. Scripts — `scripts/`

`$MOI_HOME/scripts/` holds executable helpers you keep alongside your data — backup jobs, importers,
maintenance tasks, anything that operates on the moi tree. Its reserved subfolder
`scripts/converters/` is where the per-source extractors/converters of §10 live (e.g.
`scripts/converters/joplin-to-notes`, `scripts/converters/thunderbird-to-email`).

moi places no constraint on a script's language or form — shell, Python, a compiled binary. A script
that writes into the tree is a moi tool and **MUST** follow the write rules (§7) and the relevant
conventions, like any other consumer.
