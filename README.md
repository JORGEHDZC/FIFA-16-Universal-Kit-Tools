# FIFA 16 Universal Kit Tools

**Portable kit publishing and installation for FIFA 16.**

This repository contains two companion tools:

- **FIFA 16 Universal Kit Pack Builder** — for kit makers and patch creators. It exports one or more kits from a working FIFA 16 installation into a portable `.f16kitpack` package.
- **FIFA 16 Universal Kit Installer** — for players and patch users. It installs a `.f16kitpack` into another FIFA 16 installation with preflight checks, automatic backups, database updates, dependency handling, logs, and rollback support.

The goal is simple: turn FIFA 16 kit sharing into something much closer to **plug and play**.

> [!IMPORTANT]
> **Current supported scope (F16KITPACK v1):** export kits from **Patch X** and install them into another PC using **the same Patch X / compatible database baseline**. Cross-patch migration is a future goal, not an officially supported v1 workflow.

---

## Why does this project exist?

A FIFA 16 kit is often more than one `.rx3` texture.

The FIFA 16 modding community has documented for years that simply copying a file such as `kit_TEAMID_4_0.rx3` is not enough when the corresponding kit does not exist in the database. The `teamkits` table must contain the appropriate entry for the team and kit type. Community tutorials also show that publishing a complete kit can involve RX3 files, mini-kits, database assignments, kit numbers, fonts, and sometimes patch-specific instructions. Kit-number incompatibilities have also historically produced missing/green-blue textures or crashes in some workflows.

This project automates the repeatable parts of that process and packages the data that normally has to be reconstructed manually on every user's PC.

### Community background

These tools were developed after studying the practical FIFA 16 workflows discussed by the SoccerGaming community:

- **Kits Related Questions and Answers Thread — FAQ's:** users note that an additional kit file will not work unless the corresponding kit exists in the `teamkits` database table.  
  https://soccergaming.com/forums/threads/kits-related-questions-and-answers-thread-faqs.182722/page-2
- **Hagi's Paint Shack:** a historical example of kit distribution requiring backups, RX3 copying/overwriting, additional-kit instructions, and separate handling of kit numbers.  
  https://soccergaming.com/forums/threads/hagis-paint-shack.183217/
- **Creation Master 16:** long-running community discussion of CM16 workflows, crashes, and kit-number format differences, including RevMod/FIP cases.  
  https://soccergaming.com/forums/threads/creation-master-16.183588/page-89
- **Help with kit numbers:** an example of manual kit-number troubleshooting where missing textures and even game crashes can occur.  
  https://soccergaming.com/forums/threads/help-with-kit-numbers.6468155/
- **Make Kit Numbers Larger/Smaller?:** CM16 can edit kit/short number positions and sizes, which illustrates that important visual settings live outside the shirt texture itself.  
  https://soccergaming.com/forums/threads/make-kit-numbers-larger-smaller.183629/

These references are not dependencies of the software. They are examples of the manual workflow and the problems this project is trying to reduce.

---

# Manual installation vs Universal Installer

## Traditional/manual workflow

A typical manual installation can require some or all of the following:

1. Identify the correct **team ID**.
2. Identify the correct **kit type** and **year**.
3. Back up the existing FIFA files and database.
4. Copy or replace the kit RX3 in `data\sceneassets\kit`.
5. Copy or replace the mini-kit in `data\ui\imgAssets\kits`.
6. Open Creation Master 16, DB Master, or another compatible database editor.
7. Find the correct team in the `teamkits` table.
8. If the kit is new, clone/insert a kit row.
9. Set the correct kit type and year.
10. Avoid unsafe or conflicting `teamkitid` values.
11. Recreate the original kit metadata: number font, shorts number font, colors, placements, back-name font, collar/fit information, rendering values, team colors, etc.
12. Locate and copy the correct `kitnumbers_<font>_<color>.rx3` resources.
13. Locate and copy the correct shorts-number resource when it differs from the jersey number resource.
14. Locate and copy the correct jersey back-name font (`font_<id>.ttf`).
15. Copy team-specific `specifickitnumbers_*` files when the kit uses them.
16. Save the database correctly.
17. Troubleshoot missing numbers, wrong colors, wrong placement, checker/green-blue textures, crashes, duplicate rows, or a kit that appears in CM16 but not in-game.

The exact workflow depends on the patch. Some patches introduce their own kit-number formats, Lua assignments, Revolution Mod behavior, or other conventions.

## With FIFA 16 Universal Kit Installer

For the supported workflow:

1. Run `RUN_INSTALLER.bat`.
2. Choose your FIFA 16 / patch folder the first time.
3. Select the `.f16kitpack`.
4. Review preflight.
5. Click **Install**.
6. Launch FIFA 16.

The Installer handles the package contents, database row creation/update, local IDs, kit/minikit files, number resources, fonts, verification, and backups automatically.

**No Creation Master 16 or DB Master is required on the consumer PC.**

**No FIFA regeneration step is performed or required by the supported/tested workflow.**

---

# Publishing kits manually vs Universal Kit Pack Builder

## Traditional/community publishing workflow

Without the Builder, a kit creator who wants other users to reproduce a working kit has to decide what to distribute and what to explain.

A complete release may need to include:

- the main kit RX3;
- the mini-kit;
- jersey-number RX3;
- shorts-number RX3 if different;
- jersey back-name font;
- team-specific number files;
- team ID;
- kit type;
- year;
- instructions for adding/updating the `teamkits` database row;
- number font IDs;
- number colors;
- shorts number font/color;
- number placement;
- back-name placement/layout/color;
- collar/fit/rendering values;
- backup instructions;
- special patch notes.

If any part is missing or the user reproduces the database row incorrectly, the kit can be visually wrong or fail to work.

## With FIFA 16 Universal Kit Pack Builder

The creator selects the working kit **from the source FIFA database** and the Builder does the packaging.

The output is one `.f16kitpack` containing the selected kit data and required dependencies.

Instead of publishing a folder plus a long list of database instructions, the creator can publish:

```text
Update_WC_26_Kits.f16kitpack
+
FIFA 16 Universal Kit Installer
```

The consumer selects the pack and installs it.

---

# What the Builder exports

The Builder is **DB-first**. It does not guess kit identity from loose filenames and it does not use patch-specific helper files such as `FSW\settings.ini` as the source of truth.

Authoritative source data:

```text
data\db\fifa_ng_db.db
data\db\fifa_ng_db-meta.xml
```

For each selected `teamkits` row, the Builder packages the required assets and portable metadata.

## Main assets

- Kit RX3: `data\sceneassets\kit\kit_<TEAMID>_<TYPE>_<YEAR>.rx3`
- Mini-kit DDS: `data\ui\imgAssets\kits\j<TYPE>_<TEAMID>_<YEAR>.dds`

Both are required before the kit can be added to the queue.

## Generic jersey and shorts numbers

The Builder reads the database fields that describe the kit numbers and automatically finds the referenced files:

- `numberfonttype`
- `numbercolor`
- `shortsnumberfonttype`
- `shortsnumbercolor`

From those values it collects the appropriate resources, for example:

```text
kitnumbers_60_6.rx3
kitnumbers_60_1.rx3
```

If jersey and shorts use the same number resource, the duplicate is not packaged twice.

## Jersey back-name font

The Builder reads `jerseynamefonttype` and automatically includes:

```text
data\sceneassets\jerseyfonts\font_<ID>.ttf
```

## Team-specific kit numbers

If files matching the exact team/type/year exist, they are automatically included:

```text
specifickitnumbers_<TEAMID>_<TYPE>_<YEAR>_<VARIANT>.rx3
```

## Portable `teamkits` metadata

The Builder preserves the source kit configuration needed to recreate the kit correctly on the target database, including fields related to:

- kit type;
- year;
- jersey number font;
- shorts number font;
- jersey number color;
- shorts number color;
- front number placement;
- shorts number placement;
- jersey back-name font;
- back-name case/layout/placement/color;
- jersey fit;
- collar geometry;
- rendering/detail-map settings;
- team kit colors;
- other portable values stored in the selected `teamkits` row.

The tool **does not dynamically recolor number textures**. Instead, it reproduces the source database configuration and packages the exact number/font resources referenced by that configuration.

## IDs and portability

The source `teamkitid` is stored only as provenance.

The Installer does **not** blindly copy it into the destination. It preserves an existing local `teamkitid` when updating a kit, or allocates a safe local ID when inserting a new kit.

This is important because the same patch can have different local row histories on different PCs.

---

# Builder features

- Graphical interface (Tkinter).
- English / Español first-run language selection.
- Remembers the last valid source FIFA 16 folder in local `settings.json`.
- Lets the user change the source folder at any time.
- Clears an invalid remembered path and asks again.
- Loads teams directly from the FIFA database.
- Team dropdown: no need to manually type team IDs.
- Displays the available Type/Year kit rows for the selected team.
- Multi-select kit rows.
- Multi-team pack queue.
- Add, remove, and clear queue entries.
- Optional pack name and output folder.
- Supports one kit or large multi-team update packs.
- Validates FIFA/T3DB CRCs.
- Monitors source DB and META hashes; if the source changes after loading, export is stopped until it is reloaded.
- Refuses ambiguous duplicate Team/Type/Year rows.
- Refuses kits with missing required RX3/minikit files.
- Reports exact missing team, Type/Year, and relative path.
- Refuses incomplete public packs when a referenced dependency is missing.
- SHA-256 + size metadata for every packaged file.
- Reopens the completed pack and verifies every asset after export.
- Deduplicates identical shared dependencies.
- Detailed timestamped logs.
- Export summary: teams, kits, asset references, unique files, deduplicated references, pack size, and SHA-256.
- Optional CLI mode for automation/testing.
- Portable private Python runtime; does not require a system-wide Python installation or PATH changes.
- **Never modifies the source FIFA installation.**

---

# Installer features

- Graphical interface (Tkinter).
- English / Español.
- Remembers the last valid FIFA 16 / patch folder in local `settings.json`.
- `Change FIFA 16 folder...` option.
- Invalid remembered paths are automatically discarded.
- Supports F16KITPACK format version 1.
- Verifies the package and asset SHA-256 metadata before installation.
- Validates FIFA/T3DB CRCs before and after database changes.
- Resolves the target team locally.
- Current same-patch cases normally resolve by source team ID.
- Also contains team-name resolution logic intended to help future portability work, but cross-patch installation is **not yet an officially supported v1 use case**.
- Updates existing `teamkits` rows instead of creating duplicates when the Type/Year already exists.
- Inserts missing kit rows when necessary.
- Preserves existing local `teamkitid` values when updating.
- Allocates safe local `teamkitid` values when inserting.
- Copies missing RX3/minikit files.
- Replaces different existing RX3/minikit files after backup.
- Reuses identical dependencies (`skip_same`) without touching or backing them up unnecessarily.
- Installs generic jersey/short number resources when needed.
- Installs jersey fonts when needed.
- Installs team-specific `specifickitnumbers` when present in the pack.
- Refuses to silently overwrite a shared generic dependency ID when the target contains different bytes. Automatic dependency-ID remapping is a future feature.
- Creates backups only after the user confirms installation.
- Backs up only files that will actually change.
- Backs up database/META separately under the target FIFA folder as well.
- Attempts automatic rollback if installation fails after the backup phase.
- Manual rollback support.
- LIFO-safe rollback ordering.
- Rollback refuses to overwrite files/database that have been externally changed after installation where detectable.
- Detailed logs with DB/META hashes, row counts, asset plan, DB preview, timings, and post-install verification.
- Does not require Creation Master 16 or DB Master on the consumer PC.
- Does not regenerate FIFA.
- Portable private Python runtime; does not modify system PATH.

---

# Installation of the tools

Both tools are portable Windows applications distributed as folders/ZIPs.

## First run

1. Extract the Builder and/or Installer to a normal writable folder.
2. Run:

```text
RUN_PACK_BUILDER.bat
```

or:

```text
RUN_INSTALLER.bat
```

3. If `runtime\python.exe` is not present, the launcher offers to download a private Windows x64 Python runtime.
4. Accept the runtime download.
5. The runtime is stored inside the tool's own `runtime` folder.

It does **not** install Python globally and does not add anything to PATH.

---

# How to create a `.f16kitpack`

1. Finish and test your kits in your source FIFA 16 installation first.
2. Run `RUN_PACK_BUILDER.bat`.
3. Select your language on first run.
4. Select the **SOURCE FIFA 16 / patch root**.
5. Click/load the source.
6. Select a team from the database-backed team list.
7. Select one or more Type/Year rows.
8. Add them to the Pack Queue.
9. Switch teams and continue adding kits if desired.
10. Review the queue.
11. Choose a pack name, for example:

```text
Update_WC_26_Kits
```

12. Choose an output folder if desired.
13. Create/export the pack.
14. Keep the Builder log if you plan to publish the pack.

### Recommended creator workflow

Before publishing:

- Test every selected kit in CM16/FIFA 16 on the source installation.
- Create the pack.
- Install the pack into a second clean/fresh copy of the **same patch** using the Universal Installer.
- Launch FIFA 16 and verify the kits.
- Publish the `.f16kitpack`, the supported patch/version, and the Installer.

---

# How to install a `.f16kitpack`

1. Use the patch/version required by the pack author.
2. Extract the Universal Installer.
3. Run `RUN_INSTALLER.bat`.
4. Choose your language on first run.
5. Select the FIFA 16 / patch root folder.
6. Select the `.f16kitpack`.
7. Let the Installer run **Preflight**.
8. Review the plan.
9. Click **Install**.
10. Wait for `INSTALL OK`.
11. Launch FIFA 16 and test the kits.

The last valid FIFA folder is remembered for future launches.

---

# Preflight actions explained

The Installer may classify files as:

- **copy** — the target does not have the file; it will be added.
- **replace** — the target has a different file; the old file is backed up and replaced.
- **skip_same** — the target already has exactly the same bytes; nothing is changed.

For database rows it may:

- **insert** — Type/Year does not exist for that team and a new local row is created.
- **update** — the Type/Year already exists and its portable metadata is synchronized.

---

# Rollback

The Installer creates a rollback state for files/database that actually changed.

To undo a pack:

1. Open the same Universal Installer installation used for the pack.
2. Select the same FIFA 16 folder.
3. Select the same `.f16kitpack`.
4. Click **Rollback**.

## Important rollback rule: LIFO

If you install:

```text
Pack A
Pack B
```

rollback in this order:

```text
Pack B
Pack A
```

Do not roll back Pack A while Pack B still depends on the state created after A.

The Installer also tries to protect the user from rolling back over files that were manually edited after installation.

---

# No regeneration required

The Universal Installer does **not** run a FIFA regeneration tool.

For the current supported and tested same-patch workflow, the installed kits and database changes work without a regeneration step.

If a future patch requires extra patch-specific behavior, that patch will need explicit support rather than silently adding regeneration to every installation.

---

# Example

A creator has a working World Cup patch installation with:

```text
Colombia T4/Y0
Colombia T6/Y0
Brazil T3/Y0
Senegal T3/Y0
USA T4/Y0
```

The creator selects those rows in the Builder.

The Builder reads each kit's database configuration and creates:

```text
World_Cup_Kit_Update.f16kitpack
```

The pack can contain, as required:

```text
kit RX3 files
mini-kits
jersey kitnumbers
shorts kitnumbers
jersey back-name fonts
specific kitnumbers
portable teamkits metadata
hashes and package metadata
```

The user on another PC with the supported same patch opens the Installer, chooses the pack, and clicks Install.

If a kit is already present but different, it can be updated. If an alternative kit row is missing, it can be inserted with a safe local `teamkitid`. If a dependency already matches, it is reused.

---

# Kit Type notes

Common FIFA 16 conventions are:

```text
0 = Home
1 = Away
2 = Goalkeeper
3 = Third / alternate
4+ = additional/alternate kits depending on the patch
```

Do not assume every patch uses every type in exactly the same way. The Builder reads what actually exists in the source database.

---

# Current limitations

## 1. Officially supported: same patch to same patch

For v1, the recommended/supported workflow is:

```text
Patch X / Team A / Kit N
        ↓ Builder
.f16kitpack
        ↓ Installer
Patch X / Team A / Kit N on another PC
```

The target can have a different local `teamkitid` history. That is handled locally.

## 2. Cross-patch migration is a future goal

The planned direction is:

```text
Patch X / Team A
        ↓
.f16kitpack
        ↓
Patch Y / Team A (or verified equivalent team)
```

The Installer already resolves teams locally instead of copying source IDs blindly, but true cross-patch support also needs robust handling of global dependency-ID conflicts, patch-specific kit-number formats, team equivalence, and other patch differences.

## 3. Kit deletions are not encoded in F16KITPACK v1

If the creator deletes an unwanted kit row from the source database, that deletion is **not** automatically exported.

The pack only contains explicit add/update operations.

This is intentional: automatically deleting every target kit that is absent from the creator's database would be unsafe.

A future pack schema should support explicit, reviewable removal operations.

## 4. Conflicting shared dependencies

If a generic global dependency such as `kitnumbers_<ID>_<variant>.rx3` already exists in the target under the same ID but has **different bytes**, v1 refuses to silently overwrite it.

Future versions are intended to support safe dependency-ID remapping.

## 5. Patch-specific systems

Some FIFA 16 patches use Revolution Mod, custom Lua, alternative kit-number formats, or other systems. F16KITPACK v1 focuses on the standard assets and `teamkits` data currently supported by the Builder/Installer.

---

# Troubleshooting and bug reports

If something fails, **please report it instead of manually forcing the installation first**. The logs are designed to make failures reproducible.

Include:

```text
Builder version:
Installer version:
F16KITPACK format version:
Patch name and exact version:
Source patch/version:
Target patch/version:
Was the target a fresh install?:
Pack filename:
Pack SHA-256 (if shown):
What you expected:
What happened:
Did FIFA 16 launch?:
Did the kit appear in CM16?:
Did the kit appear in-game?:
```

Attach when possible:

- the Builder log from `logs\`;
- the Installer log from `Logs\`;
- screenshot of the error/preflight;
- exact team, Type, and Year that failed.

For database-specific bugs, a maintainer may ask for the relevant `fifa_ng_db.db` and `fifa_ng_db-meta.xml`. Do not publish copyrighted game assets unnecessarily.

### Do not hide useful error information

Please do not report only:

```text
"doesn't work"
```

The most useful report is something like:

```text
Patch: Example Patch 2.1
Team: Colombia
Type: 4
Year: 0
Installer: 1.0.0
Result: Preflight refuses kitnumbers_XX_Y because target hash differs
Log attached
```

---

# Safety recommendations

- Always use the correct patch version required by the pack author.
- Do not mix unrelated patches unless you are deliberately testing unsupported cross-patch behavior.
- Let the Installer create its backup before changing files manually.
- Keep the Installer folder if you may want to rollback later.
- Keep logs for community releases.
- Test community packs on a clean copy when possible.
- Do not edit `manifest.json` manually unless you are developing/debugging the pack format.

---

# Project philosophy

The tools are not intended to replace kit creation.

You still create and test the actual shirt/short/socks textures with the tools and workflow you prefer.

The Builder + Installer solve the **distribution problem**:

> How do we take a kit that already works correctly in one FIFA 16 installation and reproduce its files + database configuration on another PC without asking every user to repeat the creator's manual CM16/database work?

That is the purpose of `.f16kitpack`.

---

# Components

## FIFA 16 Universal Kit Pack Builder v1.0.0

Creator-side tool.

```text
Working FIFA 16 installation
        ↓
Select teams/kits
        ↓
Collect assets + teamkits metadata + dependencies
        ↓
Verify + deduplicate
        ↓
.f16kitpack
```

## FIFA 16 Universal Kit Installer v1.0.0

Consumer-side tool.

```text
.f16kitpack + compatible FIFA 16 patch
        ↓
Preflight
        ↓
Backup
        ↓
Copy / replace / reuse assets
        ↓
Insert / update teamkits metadata
        ↓
Verify
        ↓
Ready to launch FIFA 16
```

---

# Status

The v1 workflow has been validated with:

- single-kit packages;
- multiple teams in one package;
- multiple new kits for one team;
- existing-kit updates;
- new `teamkits` insertion;
- local `teamkitid` allocation;
- kit/minikit copy;
- kit/minikit replacement;
- identical dependency reuse;
- generic kitnumber dependencies;
- jersey fonts;
- large community-style multi-team packs;
- rollback in controlled tests;
- fresh-install target testing.

Please continue reporting edge cases. FIFA 16 patches vary widely, and real community testing is the best way to improve future portability.
