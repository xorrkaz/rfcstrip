# Using YANG Semver Extraction (`-v`)

The `-v` flag makes `rfcstrip` name extracted YANG modules with their [YANG Semver](https://datatracker.ietf.org/doc/html/draft-ietf-netmod-yang-semver) value instead of the traditional `module@YYYY-MM-DD.yang` pattern. This gives you filenames such as `ietf-example#1.2.0.yang`, making it easy to line up generated artifacts with versioned module bundles.

## What the `-v` option does

- `rfcstrip` inspects the **first** `revision` statement in each YANG module (whether it was found in raw text, a `<CODE BEGINS>` block, or XML `sourcecode`).
- If that revision contains a `prefix:version` statement (for example `ysv:version "1.2.3"`), the parsed YANG Semver string is remembered.
- When the module is written to disk, the output file is renamed to `module-name#<semver>.yang`. Modules that already include a revision date in their filename are rewritten so the date segment becomes `#<semver>`.
- If no YANG Semver value is found, or the extracted artifact is not a YANG module, `rfcstrip` falls back to the normal filename (revision-date based or whatever was declared in `<CODE BEGINS>` / `@name`).

The version parser understands the formats allowed by YANG Semver, including optional `_non_compatible`, prerelease identifiers (e.g., `-rc.1`), and build metadata (e.g., `+build.5`).

## Running with `-v`

Use `-v` with the rest of your normal options:

```sh
# Extract every module from a text RFC into ./out using YANG Semver filenames
./rfcstrip -v -d out RFC-AAAA.txt

# Extract a single named snippet from XML and rename it with YANG Semver
./rfcstrip -v -d out -f ietf-example@2023-10-01.yang draft-example.xml
```

Tips:

- Combine `-i` to point at a directory of RFC/I-D files and `-d` to point at your desired output folder. The `-v` flag only affects the names of the generated files, not where they are stored.
- Compressed (`.gz`) inputs work exactly the same; `rfcstrip` looks inside the decompressed data when searching for `ysv:version`.

## Expected artifacts

After a successful run you should see, per YANG module:
- An extracted file named `module-name#MAJOR.MINOR.PATCH[qualifiers].yang`.
- Console output confirming the final filename, e.g.:
  ```
  out/ietf-example#1.2.0.yang: 312 lines.
  ```
- If `-v` renamed a file that originally ended in `@2024-03-01.yang`, the old file is transparently replaced with the YANG Semver-based name. You do not need to clean up the date-based filename yourself.

Artifacts for non-YANG snippets (text blocks, SMI modules, generic code) are unchanged.

## Verifying your modules expose YANG Semver

`-v` only renames files when the first revision statement contains a version extension. A minimal example:

```yang
revision 2024-03-01 {
  description "Initial release.";
  ysv:version "1.0.0";
}
```

If the YANG Semver extension (`ysv:version`) is missing, spelled differently, or placed in a later revision block, the filename remains the standard revision-date form. Adjust the source module accordingly if you want YANG Semver-aware filenames.

## Troubleshooting

- **Got the old filename anyway?** Confirm the module’s top `revision` block includes `ysv:version` (or another `prefix:version`) before any closing brace. Only the first revision is examined.
- **Multiple modules in one file?** Each YANG module is processed independently; only the modules that advertise YANG Semver are renamed.
- **Dry runs with `-n`?** Pairing `-n` with `-v` shows the final filenames in the console without writing files, allowing you to confirm the YANG Semver detection before creating artifacts.
