
UCF degree and program data is sourced from several university systems and consolidated into the **Search Service** — a Python/Django application that serves as the canonical data store for degree information. From there, data is pushed into the main WordPress website as degree posts with associated Advanced Custom Fields (ACF) metadata. The process involves a series of management commands and import steps that are run in a specific order.

---

## Overview

The data flow is roughly as follows:

```
APIM ──────────────────────────────────────────┐
Kuali Catalog ──────────────────────────────── ▼
Finance & Accounting Tuition Feed ──────► Search Service ──► WP Degree Posts
Slate (Graduate) ───────────────────────────── ▲                    │
                                               └────────────────────┘
                                             (Profile URL writeback)
```

1. Program data is pulled from APIM and written to the Search Service.
2. Catalog descriptions and catalog page URLs are pulled from Kuali and merged in.
3. Tuition and fee data is pulled from a Finance & Accounting feed.
4. Slate GUIDs for graduate programs are imported so Slate-integrated features (RFI forms, deadlines) can work.
5. Application deadlines for graduate programs are imported from Slate. Undergraduate deadlines are statically configured.
6. The refreshed data from the Search Service is imported into WordPress as degree posts.
7. The URLs of the newly created/updated WordPress degree posts are written back into the Search Service as program profiles.
8. Primary profiles are resolved for each program, distinguishing online degrees from their on-campus counterparts.

---

## Step 1: Import Programs from APIM

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-programs.py`

Program records are sourced from the **Academic Program Inventory Master (APIM)**, the university's authoritative record of all academic programs. This command creates or updates `Program` objects in the Search Service based on the APIM feed.

### Data Imported

Each program record from APIM can result in one `Program` object (for a plan) or a child `Program` object (for a subplan). The following fields are set or updated during import:

| Field | Description |
|---|---|
| `name` | The official program/subplan name |
| `plan_code` | The PeopleSoft plan code |
| `subplan_code` | The PeopleSoft subplan code, if applicable |
| `career` | Undergraduate, Graduate, or Professional |
| `degree` | The degree type abbreviation (e.g., BS, MS, CER) |
| `level` | The level classification (e.g., Bachelors, Masters, Certificate, Minor) |
| `colleges` | The college(s) the program belongs to |
| `departments` | The department(s) the program belongs to |
| `cip` | The CIP (Classification of Instructional Programs) code |
| `start_term` | The academic term in which the program began |
| `online` | Whether the program is an online program |
| `has_locations` | Whether the program has active delivery location(s) |
| `valid` | Whether the program still exists in APIM |

>[!notes]
> - Programs that existed in the Search Service but were not present in the APIM feed are marked as **invalid** at the end of the import. They are not deleted but are flagged so they can be excluded from downstream processes.
> - A college mapping file can optionally be used to override the college assignment for specific programs.
> - Online status is determined by a combination of flags in APIM, the subplan code (Z-codes), and the presence of the word "online" in the program or subplan name.
> - Pending programs (`PND`) and non-degree programs (`PRP`) are skipped.

#APIM #programs #degrees #importers

---

## Step 2: Import Catalog Descriptions from Kuali

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-catalog-descriptions.py`

Catalog descriptions and production catalog page URLs are imported from **Kuali**, the university's course and program catalog system. Each program in the Search Service is matched to its corresponding catalog entry using a fuzzy name-matching algorithm that accounts for differences in naming conventions between APIM and Kuali.

### Data Imported

| Field / Object                                    | Description                                                                  |
| ------------------------------------------------- | ---------------------------------------------------------------------------- |
| `catalog_url`                                     | The public-facing URL of the program's catalog page                          |
| `credit_hours`                                    | Total credit hours, parsed from the curriculum text                          |
| `ProgramDescription` (Catalog Description)        | A sanitized short description from the catalog                               |
| `ProgramDescription` (Full Catalog Description)   | The sanitized full description including curriculum details                  |
| `ProgramDescription` (Source Catalog Description) | The raw, unsanitized description from Kuali, stored for change detection     |
| `ProgramDescription` (Source Catalog Curriculum)  | The raw, unsanitized curriculum text from Kuali, stored for change detection |

> [!NOTES]
> - Both graduate and undergraduate catalogs are queried. Kuali "specializations" (tracks) are also imported and matched to Search Service subplans.
> - Descriptions are only re-processed if the source content has changed since the last import, unless the `--force-desc-updates` flag is used.
> - Sanitization strips disallowed HTML tags, cleans up markup, and normalizes formatting for safe display on the web.
> - The import optionally integrates with **Amazon Comprehend** for full-text analysis via an internal tool called Oscar. This step can be skipped with the `--fast` flag.
> - The import uses multithreaded processing for performance.

#Kuali #catalog #importers

---

## Step 3: Import Tuition Data

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-tuition.py`

Per-credit-hour tuition and fee rates are imported from a feed provided by **Finance and Accounting (Student Accounts)**. The command determines the appropriate fee schedule for each program based on its level and online status, then updates the program record with resident and non-resident tuition figures.

### Data Imported

| Field | Description |
|---|---|
| `resident_tuition` | Per-credit-hour tuition cost for Florida residents |
| `nonresident_tuition` | Per-credit-hour tuition cost for non-residents |
| `tuition_type` | The type of fee schedule (e.g., flat rate, per credit hour) |

> [!NOTES]
>
> - The command determines the fee schedule code based on the program's level and whether it is an online program.
> - A `TuitionOverride` model allows specific programs to be mapped to a non-default fee schedule code or skipped entirely.
> - Programs without a matching fee schedule in the feed are skipped without error.
>

#tuition #importers 

---

## Step 4: Import Slate GUIDs

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-slate-guids.py`

**Slate** is the university's CRM and admissions management platform. Each degree program managed in Slate is identified by a unique identifier (GUID). This command imports those GUIDs from the Graduate Studies Slate instance and associates them with the corresponding programs in the Search Service.

These GUIDs are prerequisites for importing application deadlines (Step 5) and are used wherever the Search Service needs to reference or link to a specific program's record in Slate, such as for Request for Information (RFI) forms.

### Data Imported

| Field | Description |
|---|---|
| Slate GUID | A unique identifier linking a Search Service program to its Slate record |

> [!NOTES]
> - Currently, only **graduate** GUIDs are imported from Slate. The undergraduate GUID import is stubbed for future implementation.
> - Programs are matched by plan code and subplan code.

#slate #importers 

---

## Step 5: Import Application Deadlines

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-program-application-deadlines.py`

Application deadline data is imported and associated with programs. Graduate deadlines are pulled from the **Graduate Studies Slate instance**; undergraduate deadlines are applied statically from configuration.

### Data Imported

| Field / Object | Description |
|---|---|
| `ApplicationDeadline` | A deadline record with admission term, career, deadline type (Domestic or International), month, and day |
| `application_requirements` | A list of application requirements associated with the program |

> [!NOTES]
> - The command is run separately for graduate and undergraduate careers using a `--graduate` flag.
> - Undergraduate deadlines use a static default list defined in the application settings (`PROGRAM_APPLICATION_DEADLINES`). The `assign_undergraduate_deadline_data` method currently applies only these defaults, with no live Slate query.
> - For graduate programs, the import queries Slate for per-program deadline data and matches records by plan code and subplan code.
> - Program-specific deadline data from Slate takes priority over any defaults previously applied.
> - Stale deadlines (those no longer associated with any program) are automatically cleaned up at the end of each run.

#slate #application #deadlines #importers 

---

## Step 6: Import Degrees into WordPress

**Repository:** [UCF-Degree-CPT-Plugin](https://github.com/UCF/UCF-Degree-CPT-Plugin)
**Command:** `includes/ucf-degree-wpcli.php` — WP-CLI command: `wp degrees import`

Once the Search Service has been populated with current data, the **UCF Degree CPT Plugin** is used to pull all of that data into WordPress. The WP-CLI command queries the Search Service API and creates or updates WordPress posts of the `degree` custom post type. Each degree post is populated with ACF fields that mirror the program data in the Search Service.

> [!NOTES]
>
> - The command requires a valid Search Service API base URL and API key, both of which are configurable via plugin options.
> - An optional `--enable_search_writebacks` flag, when set to `true`, causes data to be written back to the Search Service as each degree is imported (e.g., to record the WordPress post ID or other metadata).
> - All degree post data (titles, descriptions, tuition, deadlines, catalog URLs, etc.) flows in from the Search Service at this step.

#degrees #wordpress #importers 

---

## Step 7: Import Program Profiles (Degree Post URLs)

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-profiles.py`

After degrees have been created or updated in WordPress, this command queries the WordPress REST API to retrieve the URL of each degree post and writes it back into the Search Service as a `ProgramProfile`. This keeps the Search Service aware of where each degree lives on the public website.

### Data Imported

| Object | Description |
|---|---|
| `ProgramProfile` | A record associating a Search Service program with a URL and a profile type (e.g., the WordPress degree post) |

> [!NOTE]
> - Programs are matched by `degree_code` (plan code) and `degree_subplan_code` fields in the WordPress REST API response.
> - If a URL changes, the existing profile record is updated in place.
> - The `--remove-stale` option will delete `ProgramProfile` records for degrees that were not found in the WordPress response during the current run.
> - A profile type must be specified when running the command; this allows different types of profiles (e.g., main site, online-specific site) to be tracked independently.

#programs #search-service #importers

---

## Step 8: Process Primary Profiles

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/process-profiles.py`

Each program in the Search Service can have multiple profile URLs associated with it (e.g., one for the main UCF website and one for a UCF Online microsite). This command evaluates all programs and sets exactly one profile as the **primary profile** — the canonical URL for that program.

> [!NOTES]
> - The primary profile type is determined by a `PROGRAM_PROFILE` configuration constant, which can specify different priority rules per program.
> - A program's `primary_profile_type` field drives which of its profiles is promoted to primary.
> - All other profiles for a program are explicitly marked as non-primary.
> - This distinction is particularly important for online degrees, which may have a different primary profile URL from their on-campus equivalent.

---

## Miscellaneous Imports

The following imports are run on an as-needed basis when external data sources publish new releases, or when updated data is provided by an internal department. They are not part of the routine data refresh cycle.

---

### Import CIP Codes

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-cip.py`
**Trigger:** When NCES publishes a new CIP code release

**Classification of Instructional Programs (CIP)** codes are a taxonomy maintained by the National Center for Education Statistics (NCES) and used to classify academic programs by subject area. New CIP versions are released periodically (most recently on a 10-year cycle). When a new version is published, this command is run to load the updated codes into the Search Service.

Each `CIP` object in the Search Service maps to a specific code in a specific CIP version. Programs are associated with one or more CIPs, and those associations drive occupational data linkages (see SOC and projection data below).

#### Input File

The command accepts the NCES-published CIP CSV file directly. NCES distributes CIP data as a CSV where code values are wrapped in an Excel text-escape format (e.g., `="12.0101"`) to prevent spreadsheet applications from stripping leading zeros. The command handles this formatting automatically.

| Column | Description |
|---|---|
| `CIP Code` | The CIP code, typically in `XX.XXXX` format |
| `Title` | The official name of the CIP code |
| `Definition` | A description of the field of study |

#### Data Written

| Field | Description |
|---|---|
| `code` | The CIP code |
| `name` | The CIP title |
| `description` | The CIP definition |
| `version` | The CIP version being imported (defaults to `CIP_CURRENT_VERSION` in settings) |

>[!notes]
> - Existing CIP records are updated (name and description) if the incoming data differs; they are never deleted.
> - The `--cip-version` flag can be used to import codes under a specific version string, allowing multiple CIP versions to coexist in the Search Service.
> - Programs imported from APIM reference a specific CIP version. The version used for program imports is also controlled by `CIP_CURRENT_VERSION` in settings.

#cip #importers 

---

### Import BLS Employment Projections

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-projection-data.py`
**Trigger:** When the Bureau of Labor Statistics publishes a new Employment Projections dataset (typically every two years)

The **Bureau of Labor Statistics (BLS)** publishes national employment projections for occupational categories, including estimated job openings, employment growth, and percentage change over a 10-year horizon. This data is associated with programs in the Search Service via SOC codes. Because programs are linked to CIP codes, and CIP codes are linked to SOC codes, the projection data ultimately surfaces on the degree pages of the main website.

#### Input File

The command accepts a CSV file derived from the BLS Employment Projections dataset. The expected columns are a pre-processed subset of BLS data.

| Column | Description |
|---|---|
| `title` | The SOC occupation title, optionally followed by example job position titles |
| `code` | The SOC code |
| `begin` | Employment count at the start of the projection period (in thousands) |
| `end` | Projected employment count at the end of the projection period (in thousands) |
| `change` | Net change in employment over the projection period (in thousands) |
| `change_perc` | Percentage change in employment |
| `openings` | Projected annual job openings (in thousands) |

>[!notes]
> - Numeric values in the CSV are expected in thousands (consistent with BLS reporting). The importer converts them to full integers when storing.
> - The `title` field may encode both the occupation title and a parenthetical list of example job titles. The importer parses these apart and stores the example titles as `JobPosition` objects linked to the SOC code.
> - All existing `EmploymentProjection` records for the target report version are deleted before the new data is written.
> - `JobPosition` assignments on each SOC are cleared and fully rebuilt from the incoming data.
> - SOC codes referenced in the file must already exist in the Search Service (see [Import SOC Codes](#import-soc-codes) below). Rows referencing unknown SOC codes are skipped.
> - The `--report-version` flag controls which report version the imported projections are stored under. This defaults to `PROJ_CURRENT_REPORT` in settings.

#bls #careers #projections #importers 

---

### Import Program Outcome Data (FETPIP)

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-program-outcome-data.py`
**Trigger:** When updated FETPIP data is provided by an internal UCF department

**FETPIP (Florida Education and Training Placement Information Program)** tracks post-graduation outcomes for Florida graduates, including employment rates, continuing education rates, and average annual earnings. UCF receives periodic data exports from an internal department. This command imports that data into the Search Service and associates outcome statistics with programs by matching CIP code and degree level.

#### Input File

The command accepts a CSV file derived from the FETPIP export.

| Column | Description |
|---|---|
| `CIP` | The CIP code of the program |
| `Year` | The academic year range (e.g., `2020-2021`) |
| `Level` | The degree level (e.g., `Bachelor's`, `Master's`, `Doctorate`) |
| `Employed Full-time %` | Percentage of graduates employed full-time |
| `Continuing Education %` | Percentage of graduates continuing their education |
| `Avg Annual Earnings` | Average annual earnings of employed graduates |

#### Data Written

Each matched row creates or updates a `ProgramOutcomeStat` object and associates it with all programs that share the same CIP code and level.

| Field | Description |
|---|---|
| `employed_full_time` | Decimal representation of the full-time employment percentage |
| `continuing_education` | Decimal representation of the continuing education percentage |
| `avg_annual_earnings` | Average annual earnings as a decimal |
| `academic_year` | The `AcademicYear` object derived from the `Year` column |
| `cip` | The `CIP` object matched by the `CIP` column |

>[!notes]
> - Programs are matched by CIP code (under the current CIP version) and degree level. A single outcome row can be associated with multiple programs if several share the same CIP and level.
> - CIP codes must already exist in the Search Service. Rows with an unrecognized CIP code are skipped.
> - Level values in the CSV use degree-level terminology (e.g., `Bachelor's`, `Doctorate`). The importer normalizes minor differences (such as `Doctorate` → `Doctoral`, `Medicine` → `Professional`) before matching.
> - Rows missing a valid CIP code or year are skipped without error.
> - The `--cip-version` flag controls which version of CIP codes is used for matching.

#fetpip #ikm #importers #outcomes

---

### Import SOC Codes

**Repository:** [Search-Service-Django](https://github.com/UCF/Search-Service-Django)
**Command:** `programs/management/commands/import-soc.py`
**Trigger:** When BLS releases new Standard Occupational Classification (SOC) codes, or when an updated CIP-to-SOC crosswalk is published by NCES

**Standard Occupational Classification (SOC)** codes are a federal taxonomy for categorizing occupations, maintained by the Bureau of Labor Statistics. This command imports SOC codes and their associations to CIP codes using the official **CIP/SOC crosswalk** published by NCES. This crosswalk is what allows degree programs (identified by CIP code) to be connected to occupational data (identified by SOC code), and ultimately to the BLS employment projections.

#### Input File

The command accepts the NCES CIP/SOC crosswalk CSV.

| Column | Description |
|---|---|
| `CIP Code` | The CIP code to associate with the SOC code |
| `CIP Title` | The name of the CIP code (informational; not stored) |
| `SOC Code` | The SOC code |
| `SOC Title` | The name of the occupation |

#### Data Written

| Field | Description |
|---|---|
| `code` | The SOC code |
| `name` | The SOC title |
| `version` | The SOC version being imported |
| `cip` | Many-to-many association to the matching `CIP` object |

>[!notes]
> - CIP codes must already exist in the Search Service before running this import. Rows referencing a CIP code not found in the database are skipped.
> - Rows where the SOC Code is `NO MATCH` are skipped.
> - If a SOC code already exists (under the same version), its name is updated and the CIP association is added. Existing CIP associations are not removed.
> - The `--cip-version` and `--soc-version` flags control which CIP and SOC versions are used. These default to `CIP_CURRENT_VERSION` and the equivalent SOC version constant in settings.
> - This import should be run **after** `import-cip.py` and **before** `import-projection-data.py`, as SOC records must exist for projection data to be associated correctly.

#soc #careers #crosswalk #importers