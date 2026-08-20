# Splunk Zero to Power User Notes

## Module 1: Introduction

![Splunk Overview](https://i.vgy.me/Uzt0CB.jpg)

- **Course Structure:** Most modules consist of a lecture/concept overview followed by a hands-on Splunk Web demonstration.
- **What is Splunk?** A SIEM (Security Information and Event Management) and big data analytics platform used to collect, parse, structure, search, and visualize log data from across an enterprise network.
- **Core Functionality:** Ingests unstructured data from any source, parses raw events, and enables network analysis, threat hunting, reporting, and alerting.
- **Search Processing Language (SPL):** Splunk's proprietary query language used to search, filter, manipulate, and visualize data.

## Module 2: What Makes Up Splunk

![Splunk Architecture](https://i.vgy.me/Sc2vt6.png)

- **Core Components (The Big Three):**
  - **Forwarders:** Collects and forwards raw log data from host machines to the indexer. Types include Universal Forwarders (UF) and Heavy Forwarders (HF).
  - **Indexers:** Receives, parses, indexes, and stores raw data on disk. Organizes data into time-based directories called **buckets**. Contains compressed raw logs (`journal.z`) and index/lookup keys (`.tsidx`). Searching by time is the most efficient delimiter as it allows indexers to pinpoint exact disk locations.
  - **Search Heads:** User interface for crafting and executing SPL queries, creating dashboards, and directing search requests to indexers.
- **Deployment Architectures:**
  - **Standalone (Single-Instance):** All-in-one setup where a single server acts as Search Head, Indexer, and Input collector (common for local testing/learning).
  - **Basic Deployment:** Single Splunk server (acting as Search Head and Indexer) receives data from Universal Forwarders deployed on remote host machines.
  - **Multi-Instance Deployment:** Functional separation of roles where Search Heads, Indexers, and Forwarders run on dedicated, distinct servers for enterprise scale.
- **Clustering & Architecture Management:**
  - **Search Head Clustering (SHC):** Requires a minimum of 3 search heads (odd number for quorum/Raft consensus) to share resources and knowledge objects. Managed by a **Deployer**.
  - **Indexer Clustering:** Replicates data across multiple indexers using replication factors to ensure high availability and prevent data loss if a node fails.
  - **Deployment Server:** Centralized management component used to push configurations and updates to large fleets of forwarders.

## Module 3: Setup & Data Prep
- We are using the docker container so the installation portion of this module is skipped.
- Practice data is downloaded from the [Splunk Cloud Platform tutorial page.](https://help.splunk.com/en/splunk-cloud-platform/get-started/search-tutorial/10.5.2605/part-2-uploading-the-tutorial-data/upload-the-tutorial-data)

### Required Add-ons (via Splunkbase)
- **Splunk Add-on for Cisco WSA:** Provides parsing definitions and the `cisco:wsa:squid` source type for Cisco IronPort proxy logs.
- **Splunk Add-on for Unix and Linux:** Provides parsing definitions for Linux log files (e.g., `linux_secure`).

### Data Ingestion Mapping Summary
Data is uploaded manually via **Settings > Add Data > Upload**:

| Directory | File | Source Type | Host | Index |
|---|---|---|---|---|
| `www1` | `access.log` | `access_combined` | `web1` | `web` |
| `www1` | `secure.log` | `linux_secure` | `web1` | `security` |
| `www2` | `access.log` | `access_combined` | `web2` | `web` |
| `www2` | `secure.log` | `linux_secure` | `web2` | `security` |
| `www3` | `access.log` | `access_combined` | `web3` | `web` |
| `www3` | `secure.log` | `linux_secure` | `web3` | `security` |
| Root | `cisco_ironport_web.log` | `cisco:wsa:squid` | `cisco` | `cisco` |

### Verification
- Running `index=*` over **All time** on a clean environment yields **76,548 total events** across 3 indexes (`web`, `security`, `cisco`) and 4 hosts (`web1`, `web2`, `web3`, `cisco`).

![Splunk Setup](https://i.vgy.me/35e30c.png)

![Verification of Event Count](https://i.vgy.me/Rlichf.png)

## Module 4: Data Ingestion & Architecture

![Splunk Data Pipeline Overview](https://i.vgy.me/3yjFZt.png)

### The Data Pipeline
1. **Input:** Data is received in streams from forwarders, local log files, TCP/UDP ports, network traffic, or APIs.
2. **Parsing:** Processed by indexers where raw streams are broken into individual events, timestamps are extracted, and line breaking is applied.
3. **License Tracking:** Measures daily ingestion volume against the license quota before writing to disk.
4. **Indexing:** Transformed events are written to disk, compressed, and stored in time-based index directories (buckets).

### Core Ingestion Metadata
- **Source:** The exact path, file name, or network stream where the data originated (e.g., `/var/log/secure.log`).
- **Host:** The network hostname or IP address of the device generating or forwarding the log.
- **Source Type:** Determines how Splunk parses, formats, and breaks raw log lines into searchable event fields (e.g., `access_combined`, `linux_secure`, `csv`).

### Data Preview & Inputs
- **Data Preview:** Allows configuration verification (line breaking, timestamps, field extractions) before indexing data.
- **Input Types (Settings > Add Data):**
  - **Upload:** One-time manual file upload for testing or historical data.
  - **Monitor:** Real-time continuous collection from local/remote log files, directories, Windows Event Logs, or performance counters.
  - **Forward:** Real-time data streams routed from Universal or Heavy Forwarders.
- **Input Management:** Active monitoring inputs continuously consume license volume and can be enabled, disabled, or deleted under **Settings > Data Inputs**.

### Apps vs. Add-ons (Technology Add-ons / TAs)
- **Apps:**
  - Feature a dedicated user interface (GUI) and visual user experience.
  - Reside primarily on **Search Heads**.
  - Provide dashboards, reports, and specialized interfaces for specific vendor data or use cases (e.g., Enterprise Security, Dashboard Studio).
- **Add-ons (TAs):**
  - Background technical configurations with no dedicated GUI.
  - Deployed across **Search Heads, Indexers, or Forwarders**.
  - Provide technology-specific parsing rules, source types, input scripts, and automatic field mappings to the Common Information Model (CIM).

## Module 5: Searching & Navigation
- You can set your default application to Splunk Search & Reporting. This will allow you to access the search interface directly when you log in to Splunk.
- To set your default application, click on your username in the top right corner of the Splunk interface, select "Preferences," and then choose "Search & Reporting" as your default application.

![Click on Preferences](https://i.vgy.me/kJNnlM.png)

![Set Default Application](https://i.vgy.me/2JMWNp.png)

In the Search & Reporting section of the Splunk interface there are tabs that take you to different areas of the application, as shown below:

![Splunk Search & Reporting Tabs](https://i.vgy.me/ESJxQr.png)

- **Splunk Health & System Messages:** Check the top bar messages dropdown and the status icon (green/yellow/red) next to the username to monitor instance health.
- **Search History:** Re-populate previous searches with exact syntax and timestamps via the **Search History** dropdown directly under the search bar.
- **Time Picker:** Filter event ranges using presets, relative/real-time windows, date/time bounds, or advanced parameters (`earliest`/`latest`).
- **Search Modes:**
  - **Fast:** Prioritizes speed; suppresses non-essential field discovery and event-level details.
  - **Smart:** (Default) Toggles behavior based on query structure: acts like Fast mode when transforming commands exist (suppressing event lists) and Verbose mode when absent.
  - **Verbose:** Retrieves all possible event data and field extractions (highest disk read).
- **Timeline Controls:** Hover and drag directly across the bar chart timeline to zoom in on specific time windows.
- **Field Inspector (Left Sidebar):** View `Selected Fields` vs. `Interesting Fields`; click any field name to preview high-level value distributions.
- **Raw Event Viewing:** Expand individual log rows via the dropdown arrow to view parsed fields, raw strings, and timestamps inline.
- **Adding Items to Search:** Highlight raw string text or click field values inside an expanded event to append (`AND`) or exclude (`NOT`) them from the search string.
- **Basic Operators:**
  - **Booleans:** `AND` (implicit), `OR`, `NOT` (case-sensitive, must be capitalized). Commands and functions are case-insensitive, but Booleans must remain uppercase.
  - **Comparison:** `=`, `!=`, `<`, `>`, `<=`, `>=`.
  - **`!=` vs `NOT` Distinction:** `field!=value` matches events where the field exists and does not equal the value; `NOT field=value` returns all events that do not match the pair, including events missing the field entirely.
- **Wildcards (`*`):** Matches zero or more characters (e.g., `FAIL*` matches `fail`, `failed`, `failure`). Use sparingly as trailing/leading wildcards increase search cost.

## Module 6: Knowledge Objects (KOs)

![Knowledge Objects Overview](https://i.vgy.me/G9bCma.png)

### Definition & Purpose
- **Knowledge Objects (KOs):** Persistent user-created configurations in Splunk that enhance data analysis, enrich raw events, and facilitate collaboration across teams.
- **Examples:** Saved searches, alerts, event types, tags, field extractions, lookups, macros, and dashboards.

### Management & Naming Best Practices
- **Knowledge Manager:** Oversees KO creation, permissions, sharing, data normalization, and system performance.
- **Naming Conventions:** Recommended naming format ensures scannability and avoids object collisions:
  `[Group]_[Type]_[Description]`
  - *Example:* `SOC_Alert_ExcessiveFailedLogins`

### Permission Scopes
- **Private:** Visible and usable only by the object creator.
- **This App Only (Shared in App):** Accessible to users within the specific app where it was created (e.g., `search`).
- **All Apps (Global):** Accessible across all applications within the Splunk environment.
- **Role-Based Access Control (RBAC):** Read/Write permissions are assigned per role (e.g., Read for `Everyone`, Write restricted to `Admin`).

### Demo Highlights
- **Alert Creation:** Created an alert targeting specific event criteria (`index=security action=failure`), configured throttle settings (1 minute window), set notification actions (logging and email), and adjusted permissions to Read (`Everyone`) / Write (`Admin`).
- **Event Type Creation:** Created a custom event type via **Settings > Event Types** for `index=web action=purchase` titled `Purchases Made on Webstore`, set global scope (`All apps`), and verified the generated `eventtype` field in search results.

## Module 7: Fields & Field Discovery

### Key Value Syntax & Case Sensitivity
- **Key-Value Pairs:** Fields exist in `fieldName=fieldValue` format.
- **Case Sensitivity:** Field **names** are case-sensitive (`action` $\neq$ `Action`); field **values** are case-insensitive (`purchase` = `Purchase`).
- **Implicit Operators:** When multiple key-value pairs are listed without explicit Booleans, Splunk assumes an implicit `AND`.

### Field Sidebar & Identifiers
- **Selected Fields:** Always visible in the summary section under each expanded event in the search results (default meta fields: `host`, `source`, `sourcetype`).
- **Interesting Fields:** Extracted fields that appear in at least **20%** of the returned events.
- **Data Type Indicators:**
  - `a` = Alphanumeric field values.
  - `#` (Octothorpe) = Numeric field values.
- **Custom Selection:** Any interesting field can be promoted to a **Selected Field** by clicking it and setting `Selected: Yes`.

### Field Window Quick Reports
Clicking any field name in the sidebar opens a summary overlay detailing value distributions and quick report actions:
- **Top values:** Auto-generates a `| top <field>` stats table.
- **Top values over time:** Auto-generates a `| timechart count by <field>` visualization.
- **Rare values:** Auto-generates a `| rare <field>` statistical table.

### `!=` vs. `NOT` Logic Behavior
- **Field Does Not Equal (`field!=value`):** Evaluates strictly where the field **exists** and its value is not equal to the specified target. Returns a smaller, strict subset.
- **NOT Operator (`NOT field=value`):** Evaluates all events, returning events where the field value differs **plus** all events where the field does not exist at all.

## Module 8: Search Processing Language (SPL)

### SPL Syntax Color Coding
- **Orange (Command Modifiers & Keywords):** Boolean operators (`AND`, `OR`, `NOT`), keywords, and clauses (`AS`, `BY`).
- **Blue (Commands):** Tell Splunk what operation to perform on the dataset (e.g., `stats`, `table`, `eval`, `sort`).
- **Green (Arguments):** Variables, limits, or parameters passed to commands and functions (e.g., `limit=10`, `span=1d`, string parameters).
- **Purple/Pink (Functions):** Mathematical, evaluation, or statistical operations performed within commands (e.g., `sum()`, `avg()`, `count()`, `tostring()`).

### Search Optimization & Execution Order
1. **Filter Early (Left-to-Right Execution):** Limit data retrieved from disk first using specific metadata fields (`index`, `sourcetype`, `host`) and explicit time boundaries.
2. **Apply Calculations:** Pipe results (`|`) into transforming or evaluation commands (`stats`, `eval`).
3. **Format & Present:** Apply formatting and layout commands (`table`, `rename`) as the final stage of the pipeline.

### Core SPL Commands

| Command | Function | Example Usage |
|---|---|---|
| `table` | Renders results in a tabular layout containing only the specified fields. | `\| table _time host action status` |
| `rename` | Re-labels field names in the output for readability or reporting. | `\| rename action AS "User Action"` |
| `fields` | Explicitly keeps (`+`) or drops (`-`) specified fields to optimize internal memory usage. | `\| fields + host, status` or `\| fields - _raw` |
| `dedup` | Removes duplicate events based on matching values in specified fields. | `\| dedup clientip` |
| `sort` | Sorts results by specified fields in ascending (`+`, default) or descending (`-`) order. | `\| sort - count` |

## Module 9: Transforming Commands

### Transforming Commands Overview
- **Definition:** Commands that order search results into data tables by transforming cell values into numerical metrics for statistical analysis.
- **Smart Mode Behavior:** Automatically switches execution behavior: acts like **Fast Mode** when a transforming command is present, and **Verbose Mode** when absent.

### Core Transforming Commands

| Command | Function | Key Arguments & Options |
|---|---|---|
| `top` | Returns the most common field values. (Default: top 10). | `limit=<N>`, `showperc=true\|false`, `otherstr=<string>` |
| `rare` | Returns the least common field values. | `limit=<N>`, `showperc=true\|false` |
| `stats` | Calculates summary statistics over event fields. | Accepts `count`, `dc`, `sum`, `avg`, `min`, `max`, `list`, `values`, `BY` clauses |

### Key `stats` Functions
- **`count`:** Calculates total event count (can be grouped via `BY`).
- **`dc` / `distinct_count`:** Returns the count of unique/distinct values for a field.
- **`sum`:** Calculates the arithmetic total of numeric values across events.
- **`avg` / `min` / `max`:** Calculates average, minimum, or maximum numeric values.
- **`list`:** Lists all field values found in the events (includes duplicates).
- **`values`:** Lists unique field values found across the events (deduplicated).

### Table Formatting & Visuals
- **Field Inspector Quick Links:** Alphanumeric fields provide *Top values*, *Top values over time*, and *Rare values*. Purely numeric fields (`#`) add *Average*, *Maximum*, and *Minimum* quick links.
- **Data Table Formatting:** Use table format controls to add conditional color scales (e.g., green/yellow/red ranges), adjust number precision, or insert thousands separators.
- **Handling Missing Data:** Use `| fillnull value="<text>"` prior to statistical aggregation to replace empty/null fields with custom default values.

## Module 10: Event Correlation & Transactions

### The `transaction` Command
- **Definition:** Groups related events from single or multiple sources into a single merged event based on common field values or temporal proximity.
- **Key Fields Created:**
  - `duration`: Time difference (in seconds) between the first and last event in the transaction.
  - `eventcount`: Total number of raw events grouped within the transaction.

### Primary Command Arguments
- **`maxspan`:** Sets the maximum total time window allowed between the first and last event in a transaction (e.g., `maxspan=10m`).
- **`maxpause`:** Sets the maximum allowed time gap between consecutive individual events in a transaction (default: 1m).
- **`startswith` / `endswith`:** Defines structural boundaries for a transaction using search terms, event IDs, or field-value matches (e.g., `startswith="login"` `endswith="logout"`).

### Performance Considerations: `transaction` vs. `stats`
- **Resource & Memory Limits:** `transaction` is high-memory and computationally taxing, with a default limit of **1,000 events per transaction**. Exceeding limits silently splits transactions. Use `stats` whenever possible for high-volume datasets.
- **When to Use `stats`:** Aggregate metrics, large datasets, performant searches, or simple grouping with no limits on events per group.
- **When to Use `transaction`:** Investigating specific incident timelines, session tracking, sequence analysis (start/end boundaries), or reading grouped event bodies directly (e.g., email threads, user web browsing history via `JSESSIONID`).

## Module 11: Data Manipulation (`eval`, `where`, `search`)

### Data Filtering & Field Creation Comparison
- **`eval`:** Evaluates expressions to create new fields or overwrite existing fields in search results. It does *not* alter indexed data on disk.
- **`where`:** Filters search results using complex Boolean expressions, comparison operators, or built-in evaluation functions. Evaluates to `true` or `false`.
- **`search`:** Performs text/keyword matching or simple field-value filtering anywhere in the search pipeline (including as the first command after a pipe).

### The `eval` Command & Functions
- **Time Conversions:**
  - `strptime(string, format)`: Converts human-readable time strings into Epoch time (seconds since Jan 1, 1970).
  - `strftime(epoch, format)`: Formats Epoch timestamps into custom human-readable date/time strings (e.g., `%m/%d/%Y %H:%M`).
  - Common time variables: `%J` (day of the year, 1-366), `%H` (24-hour hour), `%M` (minute).
- **Conditional & Coalesce Logic:**
  - `case()`: Evaluates pairs of conditions and return values sequentially: `eval new_field=case(condition1, "value1", condition2, "value2")`.
  - `coalesce()`: Returns the first non-null field value from a list of field arguments: `eval user=coalesce(src_user, dest_user, "unknown")`.
- **String & Utility Functions:**
  - `md5(string)`: Computes an MD5 hash over field values.
  - Inline aggregation count: `| stats count(eval(status=404)) AS "404_Errors"`.

### `where` vs. `search` Rules
- **Pipeline Placement:** `search` can appear before or after the first pipe (`|`). `where` can *only* be used after a pipe.
- **Quote Syntax in `where`:**
  - **Double quotes (`"..."`):** Treated as string literal values (e.g., `where user="admin"`).
  - **Single quotes (`'...'`):** Treated as field references to compare two fields against each other (e.g., `where 'src_ip' == 'dest_ip'`).
- **Pattern Matching in `where`:** Uses functions like `like(field, "pattern%")` where `%` acts as a wildcard (e.g., `where like(src, "64.%")`).
- **Combining Logic:** `where` implicitly evaluates Boolean statements and does not use explicit `AND` operators in the same way as initial search clauses.

## Module 12: Field Extracting

### Field Extraction Methods Overview
- **Delimiters (Structured Data):** Used when data features consistent separators (CSV, TSV, space, pipe, semicolon).
- **Regular Expressions (Unstructured Data):** Used when logs lack rigid Delimiters or require complex pattern matching (via Field Extractor UI or inline SPL commands).
- **Extraction Timing:** Interactive Field Extractor UI and inline commands (`rex`) construct **search-time** extractions. Index-time extractions occur during ingestion parsing (`props.conf` / `transforms.conf`) and permanently alter stored data.

### Interactive Field Extractor (FX) Access Points
The Field Extractor GUI can be accessed via three primary UI locations:
1. **Settings:** `Settings > Fields > Field Extractions > Open Field Extractor`.
2. **Field Inspector:** The `Extract New Fields` link at the bottom of the left-hand field sidebar.
3. **Event Action Dropdown:** Dropping down an event row $\rightarrow$ `Event Actions` $\rightarrow$ `Extract Fields`.

### Inline Field Extraction Commands (`rex` vs. `erex`)

| Command | Usage Scenario | Key Arguments & Syntax Example |
|---|---|---|
| `rex` | Standard extraction using user-provided PCRE Regex syntax. Requires a source field (default is `_raw`). | `\| rex field=_raw "src_ip=(?<src_ip>\d+\.\d+\.\d+\.\d+)"` |
| `erex` | Automatically generates regex based on user-provided sample values. (Useful for rapid prototyping). | `\| erex target_field examples="sample_val1, sample_val2"` |

- **Named Capture Groups (`rex`):** Named regex groups `(?<field_name>pattern)` directly establish the target field name inside the extraction string.
- **`field` Argument:** Instructs `rex` or `erex` which existing field to extract data from (e.g., `field=uri_path` or default `field=_raw`).

## Module 13: Lookups

### Lookups Overview
- **Definition:** External static data sources (typically CSV files) used to enrich events with supplementary fields without re-indexing data.
- **Lookup Types:**
  - **File-based Lookups:** Static CSV files uploaded manually or placed on disk.
  - **KV Store Lookups:** Dynamic key-value store for mutable or frequently updated data (beyond core Power User scope).
  - **Automatic Lookups:** Configured under `Settings > Lookups > Automatic lookups` to automatically enrich matching events without requiring explicit `| lookup` commands in SPL queries.

### Lookup Configuration Workflow
1. **Upload File:** `Settings > Lookups > Lookup table files > New` (e.g., upload `peopleinfo.csv`).
2. **Define Lookup:** `Settings > Lookups > Lookup definitions > New` (maps the uploaded table file to a named definition for SPL reference).
3. **Set Permissions:** Update access scope (`Search App` vs. `Global`) and RBAC permissions (`Read: Everyone`, `Write: Admin`).

### Core Lookup Commands

| Command | Purpose | Basic Syntax |
|---|---|---|
| `inputlookup` | Views or searches the contents of a lookup table file directly as an event dataset. Must appear before or as a pipeline start point. | `\| inputlookup peopleinfo.csv \| where state="New York"` |
| `lookup` | Enriches incoming pipeline events by matching field values against a lookup table and pulling in additional fields. | `\| lookup productinfo.csv product_id OUTPUT description` |
| `outputlookup` | Writes search results out to a target lookup CSV file or KV store (creates or overwrites table contents). | `... \| table user_id, status \| outputlookup users.csv` |

### `OUTPUT` vs. `OUTPUTNEW`
- **`OUTPUT`:** Overwrites existing fields in the search pipeline if they share the same field name as an outputted lookup field.
- **`OUTPUTNEW`:** Prevents overwriting existing fields in the event stream; only populates target fields if they do not already exist on the event.

## Module 14: Data Visualization Fundamentals

### Overview of Visualization Types
- **Single Value:** High-visibility metric callout (e.g., total sales count, active incident counts) with configurable color thresholds (e.g., green for high sales, red for low sales).
- **Gauge:** Visual representation of progress or capacity limits.
- **Charts:**
  - **Pie Chart:** Proportional breakdown of a single-series categorical dataset (e.g., distribution of HTTP status codes or open network ports).
  - **Bar / Column Chart:** Displays discrete numeric field comparisons across categories (or over time via stacked/grouped bars).
  - **Line / Area Chart:** Displays continuous trends over time (best rendered using `timechart`).

### Charting Commands Comparison (`stats` vs. `chart` vs. `timechart`)

| Command | Time-Series Aware? | Syntax / Grouping Clauses | Primary Use Case |
|---|---|---|
| `stats` | No | `\| stats <func>(field) BY <field>` | Generates tabular aggregated data. |
| `chart` | No | `\| chart <func>(field) OVER <row_field> BY <col_field>` | Generates multi-series data tables for arbitrary categorical comparisons. |
| `timechart` | **Yes** (X-axis is implicitly `_time`) | `\| timechart [span=<time>] <func>(field) BY <field>` | Generates temporal data tables specifically for line, area, or stacked column charts over time. |

### Key Command Arguments
- **`limit=<N>`:** Controls how many distinct series values to plot (e.g., `limit=5` plots the top 5 values and groups remaining values into `OTHER`). Set `limit=0` to plot all distinct series values without truncation.
- **`useother=f`:** Suppresses the creation and display of the `OTHER` category in chart legends and plots.
- **`usenull=f`:** Excludes events containing empty or null series values from the visualization.

### Panel Formatting Options
- **Stacking Mode:** Vertical alignment of multi-series bar/column values (`Off`, `Stacked`, `Stacked 100%`).
- **Chart Overlay:** Superimposes a secondary metric line over a primary bar chart on the same shared axes (e.g., plotting total purchases as a line over total traffic bars).
- **Trellis Layout:** Splits a multi-series chart into a grid of distinct individual charts based on a common field.
- **Data Values:** Toggles numeric labels inline over data points (`On`, `Min/Max`, `Off`).

## Module 15: Visualizations, Part 2!

### Location & Mapping Commands
- **`iplocation`:** Resolves IPv4/IPv6 addresses to geographic fields (`City`, `Country`, `Region`, `lat`, `lon`) using an internal IP location database.
  - *Example:* `... | iplocation clientip`
- **`geostats`:** Generates cross-tabular statistical aggregations pre-formatted for geospatial map overlays (Cluster Maps). Requires latitude/longitude fields (e.g., `lat` and `lon`).
  - *Example:* `... | geostats latfield=lat longfield=lon count BY action`

### Statistical & Analytics Enhancement Commands

| Command | Purpose | Key Arguments & Syntax Example |
|---|---|---|
| `trendline` | Calculates moving averages over a continuous time-series field and plots an overlay line. | `\| trendline <type><period>(<field>) AS <new_field>`<br>*(Types: `SMA` = Simple Moving Avg, `EMA` = Exponential, `WMA` = Weighted)*<br>*Example:* `\| trendline SMA5(count) AS moving_avg` |
| `addtotals` | Computes column or row sums across numerical fields in a data table. | `\| addtotals row=f col=t labelfield=lon label="Total" purchase` |

### Formatting & Layout Enhancements
- **Geo-Cluster Maps:** Renders geographical distribution of events as interactive cluster bubbles; configurable for scroll-zoom behaviors and map tile provider styles.
- **Table Totals via GUI vs. SPL:** Column totals can be appended via the `addtotals` command or enabled globally through table format UI settings (`Format > Totals & Percentages`).

## Module 16: Reports & Drilldowns

### Reports Overview
- **Definition:** Saved searches that can be run on-demand or automatically executed on a set schedule.
- **Scheduling Options:**
  - **Scheduled Runs:** Runs at preset intervals (e.g., hourly, daily, weekly) or via custom **Cron syntax** (e.g., `0 6 * * 1` to run every Monday at 6:00 AM).
  - **Alert Actions:** Scheduled reports can trigger email notifications, output to CSV, or auto-generate PDF summaries.
- **Knowledge Object Management:** Saved reports are categorized as Knowledge Objects (`Settings > Searches, Reports, and Alerts`) and should adhere to standard naming conventions (e.g., `SOC_Report_ExecutablesSeen`).

### Dashboard Inputs & Tokens
- **Tokens:** Variable placeholders enclosed in dollar signs (e.g., `$loglevel$`, `$time$`) used to pass dynamic user inputs across dashboard panels, titles, or underlying search strings.
- **Common Dashboard Inputs:**
  - **Text Input:** Captures free-form user text and assigns it to a token variable (e.g., mapping text entry to `log_level=$loglevel$`).
  - **Time Picker Input:** Captures shared time boundaries (`earliest=$time.earliest$`, `latest=$time.latest$`) to globally update search timeframes across panels simultaneously.

### Drilldown Functionality
- **Definition:** Configurable click actions on dashboard panels (table cells, chart points, map nodes) that allow users to investigate underlying event details.
- **Common Drilldown Destinations:**
  - **Link to Search:** Opens a new search tab populated with the selected event's specific parameters.
  - **Link to Dashboard / Report:** Navigates to a secondary, related dashboard or report, passing click tokens as context.
  - **Set Tokens (In-Page Drilldown):** Captures click context (e.g., `$click.value$` or custom token `$userclick$`) to dynamically populate and update another panel on the *same* dashboard without leaving the page.

### Home Dashboard Configuration
- **Set as Home Dashboard:** Any user-created dashboard can be set as the landing page via `Dashboards > [Dashboard Name] > Edit > Set as Home Dashboard`.
- **User Preferences:** Update account settings (`Username > Preferences > Default Application`) to launch directly into **Home** instead of **Search & Reporting**.
- **Export Options:** Dashboards can be printed, manually exported as PDFs (`Export > Export PDF`), or scheduled for automated PDF delivery.

## Module 17: Alerts

### Alerts Overview
- **Definition:** Automated searches executed on a schedule or in real-time that trigger specific actions when results match defined criteria.
- **Purpose:** Provide immediate notification to analysts when noteworthy security or operational events occur (unlike scheduled reports which run for periodic information delivery).
- **Knowledge Object Management:** Alerts are saved Knowledge Objects (`Settings > Searches, Reports, and Alerts`), requiring proper permissions (e.g., `Shared in App` / `Global`) and standard naming conventions (e.g., `SOC_Alert_SomeoneOpenedWireshark`).

### Alert Execution & Trigger Conditions
- **Alert Type (Timing):**
  - **Scheduled:** Runs at specified intervals using preset frequency or custom Cron expressions (e.g., `* * * * *` to execute every minute).
  - **Real-Time:** Continuously evaluates incoming event streams as data is indexed.
- **Trigger Conditions:** Defines threshold logic required for an alert to fire:
  - **Number of Results:** Evaluates total count returned (e.g., `greater than 0`).
  - **Number of Hosts / Sources:** Requires a minimum number of distinct entities to match before triggering.
  - **Custom Expression:** Applies custom Boolean logic against search results.
- **Trigger Frequency:**
  - **Once:** Triggers a single alert action for the entire set of matching results.
  - **For Each Result:** Triggers an independent alert action for every individual matching event.

### Throttling & Actions
- **Throttling (Alert Suppression):** Prevents alert fatigue by suppressing subsequent notifications for a specified field value over a set duration (e.g., throttle alerts for 1 hour per `user` or `host`).
- **Severity Levels:** Categorizes risk level from `Info`, `Low`, `Medium`, `High`, to `Critical`.
- **Trigger Actions:**
  - **Add to Triggered Alerts:** Stores firing history in Splunk Web (`Activity > Triggered Alerts`).
  - **Notification Methods:** Send Email, Log Event, Webhook, Output to Lookup, or Execute a Script.

## Module 18: Welcome, Tags and Events!

### Tags Overview
- **Definition:** User-assigned labels attached to specific field-value pairs (`field=value`) to simplify searching and group disparate field values under a unified name.
- **Key Characteristics:**
  - Case-sensitive (e.g., `tag=login` $\neq$ `tag=Login`).
  - Multiple tags can be assigned to the same field-value combination.
  - Require re-running or refreshing the search after creation for the `tag` field to populate in the search results and sidebar.
- **Creation & Management:** Created inline via an expanded event's field actions menu (`Actions > Edit Tag`) or managed centrally under `Settings > Tags`.
- **Search Syntax:** Filter directly using `tag=<tag_name>` (e.g., `tag=login`).

### Event Types Overview
- **Definition:** Saved search criteria that categorize raw events based on user-defined search strings, key-value pairs, or tags.
- **Key Characteristics:**
  - Do not include time range constraints (time pickers apply at search runtime).
  - Automatically populate an `eventtype` field on all matching events.
  - Can be color-coded in Splunk Web to visually highlight critical event categories in search results (e.g., green for purchases, red for failures).
- **Creation & Management:** Saved via the search bar (`Save As > Event Type`) or configured under `Settings > Event Types`.
- **Search Syntax:** Filter directly using `eventtype=<eventtype_name>` (e.g., `eventtype=purchases_made`).

### Summary Comparison: Tags vs. Event Types

| Feature | Tags | Event Types |
|---|---|---|
| **Target** | Attached directly to a specific **field-value pair** (e.g., `EventCode=4624`). | Attached to a broader **search string / combination of filters** (e.g., `index=web action=purchase`). |
| **Generated Field** | `tag` / `tag::<field>` | `eventtype` |
| **Visual Enhancement** | Adds inline text labels next to fields. | Enables full row **color-coding** and event highlighting in the UI. |

## Module 19: Macros

### Macros Overview & Syntax
- **Definition:** Reusable, saved SPL search fragments that act as shortcuts for complex or repetitive queries.
- **Benefits:** Minimizes search-string errors, standardizes report logic, simplifies complex mathematical evaluations, and speeds up daily workflows.
- **Execution Syntax:** Must be enclosed in backticks (`` `macro_name` ``). *Note: Backticks (`` ` ``) are distinct from single quotes (`'`).*
- **Search Expansion Shortcut:**
  - **Windows:** `Ctrl` + `Shift` + `E`
  - **macOS:** `Cmd` + `Shift` + `E`
  - Displays the full, unrolled SPL search string executing behind the macro.

### Macro Arguments & Naming Conventions
- **Passing Parameters:** Macros can accept one or more user-defined positional arguments enclosed in parentheses (e.g., `` `loglevel(error)` ``).
- **Naming Rule:** When defining a macro that uses arguments, the total number of arguments **must** be explicitly appended to the macro title in parentheses under settings:
  - *Format:* `macro_name(number_of_args)` (e.g., `loglevel(1)`).
- **Argument Placeholders:** Arguments inside the macro definition string are enclosed in dollar signs (e.g., `log_level=$input$`).

### Configuration Navigation
- **Create/Manage Macros:** `Settings > Advanced Search > Search Macros > Add New`.
- **Permissions:** Saved as Knowledge Objects; can be scoped to `This App Only` or `All Apps` (Global) with RBAC permissions (`Read: Everyone`, `Write: Admin`).

## Module 20: Workflows to Save You Time

### Workflow Actions Overview
- **Definition:** Custom context-menu options integrated into Splunk Web that allow users to interact with external web resources, push data to third-party applications, or execute secondary Splunk searches directly from raw events or field menus.
- **Types of Workflow Actions:**
  - **`GET` Actions:** Sends field values via HTTP GET parameters to external web resources for quick analytical lookups (e.g., querying IP Whois, Threat Intelligence databases, or Windows Event Code definitions).
  - **`POST` Actions:** Submits field values via HTTP POST payload to an external web endpoint or API (e.g., automatically generating a ServiceNow ticket, submitting a Security Incident report, or filling out web forms).
  - **`Search` Actions:** Launches a secondary, targeted Splunk search in a new or existing window based on specific field values contained within the current event.

### Key Configuration Parameters (`Settings > Fields > Workflow actions`)
- **Action Name:** Unique internal identifier for the workflow object.
- **Label:** User-facing text rendered in the event context menu (supports field variables enclosed in dollar signs, e.g., `Whois Lookup for $clientip$`).
- **Apply to Fields:** Restricts visibility of the workflow action strictly to events containing specified fields (e.g., `clientip`, `src_ip`, `event_code`).
- **Show Action In:** Determines menu placement (`Event Menu`, `Fields Menu`, or `Both`).
- **URI Template:** The target web address including variable interpolation enclosed in dollar signs (e.g., `https://www.domaintools.com/research/whois/$clientip$`).

### Usage Context & Best Practices
- **Menu Access:** Executed by expanding an event row and selecting the custom action from the **Event Actions** dropdown menu.
- **Efficiency:** Drastically reduces investigation time for SOC analysts by eliminating manual copy-pasting of indicators (IPs, Hashes, Domains) into external lookup sites.

## Module 21: Data Normalization & Troubleshooting

### Field Aliases & Data Normalization
- **Purpose:** Normalizes dataset fields across diverse source types by mapping different vendor field names to a single standardized name (e.g., mapping `src`, `clientip`, and `ip_address` to `source_ip`).
- **Behavior:** The original field remains stored on disk and visible in the Field Inspector alongside the newly created alias.
- **Configuration:** `Settings > Fields > Field aliases > Add new`. Specify the target `sourcetype`, the existing field name, and the alias name.

### Calculated Fields
- **Purpose:** Evaluates `eval` functions automatically at search runtime to populate a new field based on existing field values, avoiding the need to write manual `eval` statements in every search string.
- **Configuration:** `Settings > Fields > Calculated fields > Add new`. Define the target `sourcetype`, name the output field, and input the `eval` expression (e.g., `bytes / 1024 / 1024`).

### Index Bucket Lifecycle & Storage
Splunk stores indexed event data on disk in time-based directories called **buckets**:

| Bucket State | Description & Searchability |
|---|---|
| **Hot** | **Only writable bucket.** Actively receives newly indexed incoming log streams. Rolled to Warm based on size (`maxDataSizeMB`), max age, or Splunk service restart. Fully searchable. |
| **Warm** | Read-only. Rolled over from Hot when size or time thresholds are met. Fully searchable. |
| **Cold** | Read-only. Rolled over from Warm as data ages; typically moved to cheaper storage tiers. Fully searchable. |
| **Frozen** | Archived to external storage or deleted based on retention policy. **Not searchable.** |
| **Thawed** | Restored data moved back from Frozen storage into Splunk to enable searching. |

### Troubleshooting & Search Performance Tools
- **Job Inspector:** Diagnostics tool opened via `Activity > Jobs` or `Job > Inspect Job`. Provides detailed execution metrics (time spent per search component, event count, and disk read cost).
- **`dbinspect` Command:** Directly inspects index bucket metadata (e.g., `... | dbinspect index=* | table bucketId, state, index`).
- **SPL Editor Preferences:** Configured under `Username > Preferences > SPL Editor` (`Full`, `Compact`, or `None`) to adjust auto-completion, syntax guidance, and inline documentation hints.

## Module 22: Datamodels

### Datamodels Overview & Acceleration
- **Definition:** A hierarchically structured search-time mapping of semantic knowledge over one or more datasets (composed of parent and child datasets).
- **Purpose:** Organizes unstructured log streams into standardized schemas and enables **Datamodel Acceleration**.
- **Acceleration Mechanism:** When accelerated, Splunk pre-computes data summaries and writes them to high-performance index files (`.tsidx`). Queries executed against accelerated datamodels bypass raw disk parsing, enabling drastic search speed improvements over massive datasets.

### Datamodel Structure
- **Parent Datasets:** Base datasets representing broad categories (e.g., `Authentication`, `Web`, `Network Traffic`).
- **Child Datasets:** Subsets that inherit all attributes/fields of the parent dataset while applying additional, refined filters (e.g., `Authentication > Successful Authentication`).

### Pivot Tool
- **Definition:** A visual drag-and-drop interface within Splunk Web that allows non-technical users to build charts, visualizations, and summary reports directly from Datamodels without typing SPL.

### Searching Datamodels via SPL

| Command / Technique | Function | Example Syntax |
|---|---|---|
| `datamodel` | Directly searches existing Datamodel definitions, fields, and dataset structures. | `\| datamodel Web access_combined search` |
| `tstats` | Extremely fast statistical command that queries summary index files (`.tsidx`) of accelerated Datamodels. Fully qualified field names required (e.g., `Web.action`). | `\| tstats count FROM datamodel=Web BY Web.action` |

### Key Differences: `stats` vs. `tstats`
- **`stats`:** Scans and parses raw event data retrieved from index buckets on disk (slower on massive datasets).
- **`tstats`:** Summarizes metric data strictly from indexed `.tsidx` summary files of accelerated Datamodels or data metrics (exponentially faster).

## Module 23: The Common Information Model (CIM)

### Common Information Model Overview
- **Definition:** A standardized schema framework containing 22 pre-configured Data Models used to normalize field names and event structures across disparate vendor log sources.
- **Primary Purpose:** Normalizes vendor-specific field names (e.g., `clientip`, `src`, `ip_address`) into consistent, standard CIM field names (e.g., `src_ip`) to enable seamless multi-source correlation.
- **Enterprise Security (ES) Compliance:** Mandatory for Splunk Enterprise Security and other premium apps, which rely strictly on CIM-compliant data models and field definitions to drive correlation rules.

### Data Normalization & CIM Mapping
- **CIM Requirements:** Events require two main elements to align with a CIM Data Model:
  1. **CIM Tags:** Event types or tags attached to raw events matching the model's base criteria (e.g., `tag=web`).
  2. **Normalized Fields:** Field aliases, extractions, or calculated fields that rename raw fields to standard CIM names (e.g., `useragent` $\rightarrow$ `http_user_agent`, `clientip` $\rightarrow$ `src`).

### Splunk Add-on Builder (TA Builder)
- **Purpose:** A GUI tool used to create custom Technology Add-ons (prefixed with `TA_`) to onboard and map data to CIM Data Models.
- **Workflow:** Import source type (`access_combined`) $\rightarrow$ Map to target CIM Data Model (`Web`) $\rightarrow$ Map raw fields to green/matching CIM target fields via Field Aliases $\rightarrow$ Validate and package.