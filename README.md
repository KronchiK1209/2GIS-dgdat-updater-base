HELP ME PLEASE DONATION https://www.donationalerts.com/r/kronchik_1209 КТО МОЖЕТ ПОДДЕРЖИТЕ ДОНАТОМ https://www.donationalerts.com/r/kronchik_1209 У МЕНЯ СКОРО СВАДЬБА, ПОЭТОМУ ВЫКЛАДЫВАЮ В ОБЩИЙ ДОСТУП, А КТО СМОЖЕТ ПОДДЕРЖИТЕ МЕНЯ, ПРОЕКТ И РАЗВИТИЕ УТИЛИТ, ДАЛЕЕ ХОЧУ СДЕЛАТЬ АКТУАЛИЗАЦИЮ ИЗ ЯНДЕКСА ВСЕХ МЕСТ, НО НУЖНА ВАША ПОДДЕРЖКА. СПАСИБО ВСЕМ, КТО ОТКЛИКНУЛСЯ!

# DGDat Date Lab Updater

**Offline GDF/DGDat header analyzer and metadata date updater — a single-file HTML tool with no external dependencies.**

DGDat Date Lab Updater is a browser-based laboratory utility for examining supported GDF (`.dgdat`) headers, identifying a CRC-protected header region, locating a set of date-like fields, recalculating header CRC32 values, and exporting a separate modified copy.

The project is intended for format research, reproducible experiments, and educational use. It does not download, update, rebuild, or validate the actual map content.

> **Important:** Changing dates in a database header does not update roads, addresses, buildings, POIs, or any other map data. A file whose metadata date was changed must not be treated as a current map or used for operational navigation, dispatch, emergency response, or other safety-critical decisions.

## Project status

- Current version: **3.0**
- Application type: standalone HTML + vanilla JavaScript
- Dependencies: none
- Network access: not required
- Build step: none
- Processing: local in the user's browser
- Input: `.dgdat` file
- Output: a separate `.dgdat` file, a discovered-profile JSON file, and a diagnostic report JSON file

Version 3.0 supports automatic discovery within a **specific observed GDF header family**. It is not a universal parser for every historical or future 2GIS/DGDat format.

## Features

### 1. Automatic header-boundary discovery

The application:

1. Checks the GDF signature `07 47 44 46`.
2. Reads the stored outer and inner CRC32 fields at their observed common offsets.
3. Scans the beginning of the file (up to 64 KiB) for a unique header end where both CRC calculations match their stored values.
4. Refuses to continue if no unique boundary is found or if multiple boundaries match.

The current implementation assumes these common CRC field locations:

| Field | Offset | CRC input begins at |
|---|---:|---:|
| Outer CRC32 | `0x0E` | `0x12` |
| Inner CRC32 | `0x15` | `0x19` |

The discovered end offset is exclusive. In observed reference samples it was `0x154` for Norilsk 142/163 and `0x1A0` for Krasnoyarsk 250.0.0. These are reference observations, not a hard-coded city list.

### 2. Date-field candidate discovery

Within the CRC-protected header region, the application looks for values encoded as:

```text
C0 + uint16 big-endian
```

The 16-bit value is interpreted as a day serial relative to `1899-12-30`.

Current candidate-search rules:

- Candidate dates are limited to the range 2000-01-01 through 2100-01-01.
- Equal serial values are grouped together.
- The utility searches for a unique, ordered cluster of four different date values spanning no more than 550 days.
- The roles are inferred by ascending date order:
  - `bd` — earliest date
  - `uad` — next date
  - `ewd` — next date
  - `ed` — latest date
- Each selected value must occur at exactly one byte offset.
- The user must explicitly confirm the inferred mapping before export.

These rules are heuristics based on the examined samples. They do not establish a complete formal specification of GDF.

### 3. CRC32 recalculation

The application uses CRC-32/IEEE (the same CRC family commonly exposed as `zlib.crc32`).

When a confirmed header is modified, CRC values are recalculated in this order:

1. Inner CRC32.
2. Outer CRC32.

Both values are stored in big-endian byte order. The outer CRC calculation includes the updated inner CRC field, which is why calculation order matters.

### 4. Separate output file

The original input file is never modified in place.

The application changes only:

- the four selected date fields;
- the two CRC32 fields.

It checks that:

- the output CRC fields match fresh calculations;
- date values can be read back;
- no other byte within the discovered header boundary was changed;
- the output file size matches the input size.

The output is assembled as a new `Blob` from the modified header and a `File.slice()` of the original payload. The map payload is copied as-is; it is not decompressed, decoded, or rebuilt.

### 5. Large-file handling

The application reads only the beginning of the file (up to 64 KiB) for discovery and header editing.

The remaining file content is passed through as a slice of the original `File`, rather than being loaded into one large JavaScript `ArrayBuffer`. This reduces memory pressure when handling large `.dgdat` files.

Actual browser and operating-system download limits still apply.

### 6. Diagnostic exports

The utility can export:

- a discovered-profile JSON file containing the offsets, date serials, deltas, and CRC boundary;
- a report JSON file describing the detected input, output metadata, date mapping, CRC ranges, and checks performed.

## Quick start

1. Download `DGDat_Date_Lab_Updater_v3.html`.
2. Open it in a current desktop browser (for example, Chrome, Edge, or Yandex Browser).
3. Select the original `.dgdat` file.
4. Wait for the GDF signature, CRC boundary, and date candidates to be examined.
5. Review the detected date candidates and inferred roles.
6. Select the new `bd` date.
7. Confirm the detected date mapping.
8. Click **“Сформировать новую DGDat”**.
9. Keep the original file unchanged and test the generated copy separately.
10. Save the JSON profile and report if you need a record of the experiment.

No local server is required. The utility does not need an internet connection.

## Repository contents

Suggested minimal repository layout:

```text
dgdat-date-lab-updater/
├── DGDat_Date_Lab_Updater_v3.html
├── README.md
├── LICENSE
└── .gitignore
```

The application is intentionally distributed as a single HTML file so it can be opened locally without installing packages or setting up a build environment.

## Technical format notes

The current implementation relies on the following observed envelope:

| Item | Current assumption |
|---|---|
| File signature | `07 47 44 46` (`GDF`) |
| Outer CRC field | `0x0E`, 4 bytes, big-endian |
| Outer CRC input | `[0x12, headerEnd)` |
| Inner CRC field | `0x15`, 4 bytes, big-endian |
| Inner CRC input | `[0x19, headerEnd)` |
| CRC algorithm | CRC-32/IEEE |
| CRC update order | inner, then outer |
| Date marker | `0xC0` |
| Date value | unsigned 16-bit, big-endian |
| Date epoch | `1899-12-30` |
| Discovery scan limit | first 64 KiB |
| Candidate-date window | 2000-01-01 to 2100-01-01 |
| Four-date cluster span | at most 550 days |

Observed sample boundaries:

| Sample family | Observed header end |
|---|---:|
| Norilsk 142.0.0 / 163.0.0 | `0x154` |
| Krasnoyarsk 250.0.0 | `0x1A0` |

These observations should be treated as test fixtures, not proof that all files with the same city name or version use the same layout.

## Safety and data-integrity model

The application is designed to fail closed:

- Unknown GDF signature → stop.
- CRC mismatch → stop.
- No unique CRC boundary → stop.
- No unique four-date cluster → stop.
- Repeated candidate values at multiple offsets → stop.
- Invalid date or 16-bit overflow → stop.
- Unexpected changes in the protected header → stop.

The utility does not patch `grym.exe`, bypass application checks, or attempt to disable compatibility or integrity validation in the 2GIS loader.

A successful local CRC check only proves that the header matches the implemented CRC calculation. It does **not** prove that the full database is valid, compatible with a particular loader, current, or suitable for navigation.

## Limitations and future work

Version 3.0 does not:

- parse every known or unknown DGDat generation;
- identify arbitrary date encodings;
- discover arbitrary CRC algorithms, field offsets, byte order, or CRC input ranges;
- decode or validate every proprietary map-data block;
- update map content or download new geographic data;
- confirm compatibility by launching the target Windows application;
- guarantee that a modified database will be accepted by every 2GIS release.

For a new format family, collect a known-good original sample and compare it with another sample from the same family. Extend the detector only after confirming the header structure, date semantics, CRC parameters, and loader behaviour. Do not add a guessed profile merely because a byte pattern resembles a date.

## Privacy

All processing happens locally in the browser:

- no upload endpoint is used;
- no analytics or telemetry are included;
- no third-party JavaScript libraries are loaded;
- no network connection is required.

The selected database remains on the user's device. The browser creates a local download for the modified copy and JSON diagnostics.

## Legal and project scope

This is an independent research utility. It is not an official 2GIS product and is not affiliated with or endorsed by 2GIS.

Do not commit commercial `.dgdat` databases, proprietary executables, city map payloads, credentials, or other copyrighted or confidential material to the repository unless you have the rights and authorization to distribute them.

The repository should contain the tool source and documentation, not sample map databases.

## Contributing

Useful contributions include:

- new verified header fixtures (small sanitized headers only, where redistribution is permitted);
- reproducible CRC test cases;
- parser improvements with documented evidence;
- regression tests for known layouts;
- UI and accessibility improvements;
- documentation corrections.

Every new automatic profile should include:

1. a known-good input sample or a safely shareable header fixture;
2. exact date-field offsets and encoding;
3. exact CRC field offsets, ranges, algorithm, and byte order;
4. a reproducible expected output header;
5. a statement of what was tested in the actual loader, if loader compatibility is claimed.

Never claim loader compatibility based on CRC checks alone.

## License

Choose and add a license before publishing. If you want permissive reuse, the MIT License is a common option, but make sure you have the right to license all code and included assets. Do not add a license file until you have selected the intended terms.
