# Unit Profile Sync Contributor Guide

## Project Contract

- This is Moodle local plugin `local_unit`, installed at `local/unit`.
- The plugin imports the bundled WDARFF and ARNG CSV files into `{local_unit_orgs}` and syncs Moodle user profile fields from the `unit_uic` custom profile field.
- UIC values are trimmed and converted to uppercase before lookup. Invalid or empty UICs must clear `unit_uic`, `department`, and `institution`.
- Treat the bundled CSV files as source data. Do not edit, remove, rename, or regenerate them without an explicit source-data update requirement and documentation.

## Profile Mapping

- `NAMEPATH` is a `>`-delimited hierarchy. Ignore empty segments created by its leading delimiter.
- The final `NAMEPATH` segment is the selected UIC's name and is synced to Moodle `department`.
- The segment immediately before it is the parent UIC's name and is synced to Moodle `institution`.
- Do not use `DRRSANAME` for either field. Do not derive the parent name by looking up a separate row when `NAMEPATH` supplies it.
- If the path has no parent segment, leave `institution` empty. If it has no final segment, leave `department` empty.

## Data And Schema Changes

- Keep `db/install.xml` and `db/upgrade.php` consistent for every schema change.
- Add an ordered, idempotent upgrade step and `upgrade_plugin_savepoint()` for installed-site schema changes.
- Bump `$plugin->version` when adding an upgrade step or when an update must automatically reimport bundled CSV data.
- CSV imports identify records by normalized `UIC`; preserve this upsert behavior unless a migration is explicitly required.

## Implementation Rules

- Follow Moodle PHP conventions and retain `defined('MOODLE_INTERNAL') || die();` in PHP entry points.
- Keep changes minimal and preserve the custom profile field shortname `unit_uic` and table name `local_unit_orgs`.
- Update `README.md` for changed operator-facing behavior and add a concise entry under `Unreleased` in `CHANGELOG.md`.
- Do not add fallback mappings that silently return to `DRRSANAME`.

## Verification

- Run `php -l` on changed PHP files.
- Run `git diff --check`.
- When a Moodle installation is available, run `php local/unit/cli/import.php` after importer/data changes and `php local/unit/cli/sync.php` after profile-sync changes.
- Verify representative UICs against their source `NAMEPATH`, including a leaf UIC and a root UIC.
