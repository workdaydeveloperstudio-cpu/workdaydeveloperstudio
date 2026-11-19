# WORKDAY STUDIO MASTER ARCHITECTURE, DESIGN PATTERN & DIAGRAM KNOWLEDGE BASE
**Author:** Yogarajan Venkataraman (YV)  
**Standard:** Enterprise Workday Studio Development, Eclipse GMF/EMF Notation & Governance Standards  
**Status:** Living Canonical Master Specification  
**Version:** 5.0 (Consolidated Master)

---

## 1. Core Architecture & Eclipse GMF/EMF Contract

Workday Studio integrations rely on an unyielding dual-file contract between the Spring runtime descriptor (`assembly.xml`) and the Eclipse Graphical Modeling Framework (GMF) / Eclipse Modeling Framework (EMF) notation descriptor (`assembly-diagram.xml`).

```
┌──────────────────────────────────────────────┐       ┌──────────────────────────────────────────────┐
│                 assembly.xml                 │       │             assembly-diagram.xml             │
│        (Spring / CapeClear Runtime)          │       │            (Eclipse GMF/EMF Model)           │
├──────────────────────────────────────────────┤       ├──────────────────────────────────────────────┤
│ • Declares beans, services, mediators, routes│ ◄───► │ • Provides 2D spatial coords (x, y)          │
│ • Canonical IDs: Start_YV, SplitRows_YV, etc.│  1:1  │ • References beans via href="assembly.xml#ID"│
│ • Defines logical routing & error branches   │ Exact │ • Maps visual hierarchy into <swimlanes>     │
│ • Enforces sequential/parallel execution     │ Match │ • Defines layout anchors & visual routing    │
└──────────────────────────────────────────────┘       └──────────────────────────────────────────────┘
```

### 1.1 The Dual-File 1:1 Synchronization Rule
1. **Zero Phantom Nodes:** Every `<visualProperties>` entry in `assembly-diagram.xml` must map to a component ID or valid EMF XPath fragment physically present in `assembly.xml`.
2. **Zero Orphaned Runtime Beans:** Every top-level mediator, service, route, splitter, or aggregator declared in `assembly.xml` must have a corresponding `<visualProperties>` node in `assembly-diagram.xml`.
3. **Exact ID Matching:** Component IDs are strictly case-sensitive. Never use guessed or placeholder IDs.
4. **Enclosure Guarantee:** Every visual element in `<visualProperties>` must be mapped inside a `<swimlanes>` element. Nodes floating outside swimlanes trigger GMF canvas auto-rescaling and coordinate distortion.

### 1.2 Preventing `Scala.MatchError` & EMF Proxy Corruption
When Workday Studio loads `assembly-diagram.xml`, its internal parser expects every EMF URI fragment to resolve against the in-memory AST. If a referenced fragment does not exist or has an unexpected type:
- Eclipse displays red 'X' decorators across the canvas.
- The Graphical Assembly Editor crashes with `scala.MatchError` or `NullPointerException`.
- Studio enters an unrecoverable corrupted visual state.

**Remediation Rule:**
- For named beans: Always use `assembly.xml#ComponentId` (e.g., `href="assembly.xml#Start_YV"`).
- For anonymous beans (routes/sub-routes/splitters): Use exact Spring mixed-content ordinals (e.g., `assembly.xml#//@beans/@mixed.1/@route.0/@subRoute.0`).
- Never guess mixed-node numbers from memory or copy them blindly between projects.

---

## 2. Canonical Diagram Design Patterns (INT0016 Standard)

The reference benchmark for Studio diagrams is the canonical **INT0016 architecture**. 

### 2.1 The Flat Top-Level Hierarchy
* **Anti-Pattern:** Wrapping all functional stages inside an outer vertical "mega-swimlane" (e.g., an outer container spanning $x=10, y=10$ with width $>2500\text{px}$). This causes Studio's layout engine to apply dynamic scaling, shifting origins and breaking orthogonal lines.
* **Canonical Pattern:** Every primary functional stage is an independent top-level horizontal swimlane placed directly under `<wdnm:Diagram>`.

```
CANONICAL ROOT PATTERN:
<wdnm:Diagram xmlns:wdnm="http://workday.com/studio/editors/notation">
  <element href="assembly.xml#WorkdayAssembly"/>

  <!-- Top-Level Horizontal Tiers (Root Level) -->
  <swimlanes x="20" y="20"   name="1 - Entry and Initialize" ... />
  <swimlanes x="20" y="300"  name="2 - Document Retrieval" ... />
  <swimlanes x="20" y="630"  name="3 - Parse CSV and Retrieve Report" ... />
  <swimlanes x="20" y="1120" name="4 - Validate Records and Route" ... />
  <swimlanes x="20" y="1520" name="5 - Map and SOAP Call" ... />
  <swimlanes x="20" y="1840" name="6 - Process Response and Row Faults" ... />
  <swimlanes x="20" y="2140" name="7 - Summary and Final Status" ... />

  <!-- Dedicated Top-Right Error Handler (Root Level) -->
  <swimlanes x="1860" y="20" name="YV Shared Error Handlers" orientation="VERTICAL" ... />
</wdnm:Diagram>
```

### 2.2 Strict Zero-Based Swimlane Index Sequencing
Studio’s GMF runtime resolves nested sub-swimlanes using **0-based document order indices** (`#//@swimlanes.N`):
1. The parser reads all `<swimlanes>` tags in sequence from top to bottom (Index 0, Index 1, Index 2, ...).
2. Child swimlane membership **must** be declared using child elements:
   ```xml
   <!-- VALID INT0016 NESTING SYNTAX -->
   <swimlanes x="20" y="1120" name="4 - Validate Records and Route">
     <elements href="#//@swimlanes.8"/>
     <elements href="#//@swimlanes.9"/>
     <elements href="#//@swimlanes.10"/>
   </swimlanes>
   ```
3. **Strict Ban on String Attributes:** Never use the deprecated `elements="//@swimlanes.8 //@swimlanes.9"` attribute on parent swimlane tags. It breaks EMF index binding.

### 2.3 Paired Vertical Sub-Swimlanes for Async-Mediation Error Handling
When an `async-mediation` contains an internal `<cc:send-error>` that routes to a `PutIntegrationMessage` (PIM) or local logger:
- Group both components inside a dedicated vertical sub-swimlane (`orientation="VERTICAL"`).
- Place the error logger directly beneath the mediator ($+110\text{px}$ to $+130\text{px}$ vertical offset).
- This visually documents the error boundary without crossing wires over main flow paths.

```
┌─────────────────────────────────┐
│ Swimlane (orientation="VERTICAL")│
│  ┌───────────────────────────┐  │
│  │     AsyncMediation        │  │
│  └─────────────┬─────────────┘  │
│                │ send-error     │
│                ▼                │
│  ┌───────────────────────────┐  │
│  │   PutIntegrationMessage   │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

### 2.4 The Top-Right Isolated Error Tier
Global and fatal error handlers must never be placed inside the main business flow:
- Position `Global Error Handler`, `CriticalError_YV`, and `CriticalPIM_YV` at the top right of the canvas (`x="1610"` to `x="1900"`, `y="20"` to `y="100"`).
- This establishes clear separation between happy-path orchestration and catastrophic failure handling.

### 2.5 Group 4 Multi-Column Pattern (3-Column Processing)
For batch integrations (e.g., INT0099, INT0016) that validate, loop, and route records:
- Group 4 must be structured into three side-by-side vertical sub-swimlanes:
  - **Column 4A (`x=40–60`):** Format, schema, and tenant duplicate validation.
  - **Column 4B (`x=580–680`):** Sequential loop controller (`Splitter` and accumulator).
  - **Column 4C (`x=1220–1320`):** Row classification, business rule routing, and per-row PIM reporting.

---

## 3. Coordinate System & Orthogonal Routing Cadence

### 3.1 Spatial Layout Cadence
Maintain consistent visual spacing to prevent overlapping connectors:
- **Horizontal Step:** $\Delta x = 180\text{px}$ to $240\text{px}$ between sequential components.
- **Vertical Tier Spacing:** $\Delta y = 300\text{px}$ to $350\text{px}$ between major horizontal swimlanes.
- **Branch / Fault Step-Down:** $\Delta y = +110\text{px}$ to $+130\text{px}$ for sub-routes, error handlers, and alternate choice routes.

### 3.2 Normalized Connection Anchors
Explicit connection anchors ensure orthogonal, non-overlapping route lines:
- **Left-to-Right Flow:** `sourceAnchorX="1.0" sourceAnchorY="0.5"` to `targetAnchorX="0.0" targetAnchorY="0.5"`
- **Vertical Step-Down:** `sourceAnchorX="0.5" sourceAnchorY="1.0"` to `targetAnchorX="0.5" targetAnchorY="0.0"`
- **Response Routing:** `routesResponseTo` must connect back cleanly to the calling mediation tier.

---

## 4. XSLT 2.0 & Workday Integration Functions

### 4.1 QName Space Restriction in `<xsl:param>`
In Saxon/Workday Studio XSLT 2.0, parameter names must be valid XML QNames.
- **Forbidden:** `<xsl:param name="inmap.Bank Account"/>` or `<xsl:param name="inmap.Ledger Account"/>`. Parameter names with spaces trigger fatal compilation errors.
- **Standard:** Use inline Workday integration functions:
  ```xml
  <xsl:value-of select="is:getIntegrationMapValue('Bank Account', $rawCompany)"/>
  ```

### 4.2 Fail-Closed Architectural Design
Data transformation stylesheets must terminate processing if upstream contracts fail:
```xml
<!-- Example from normalize_existing_emails_YV.xsl -->
<xsl:if test="count(/*[local-name()='Report_Data']/*[local-name()='Report_Entry']) = 0 and not(/*[local-name()='Report_Data'])">
  <xsl:message terminate="yes">
    FATAL: Upstream RaaS Report did not return valid Report_Data. Failing closed to prevent unauthorized duplicate loads.
  </xsl:message>
</xsl:if>
```

### 4.3 High-Performance Intra-File Deduplication
Use XSLT 2.0 keys for $O(1)$ duplicate lookups across large CSV/XML inputs:
```xml
<xsl:key name="email-by-row" match="Record" use="lower-case(normalize-space(Email))"/>

<!-- Detection of intra-file duplicates -->
<xsl:variable name="email" select="lower-case(normalize-space(Email))"/>
<xsl:variable name="isDuplicate" select="generate-id(.) != generate-id(key('email-by-row', $email)[1])"/>
```

---

## 5. Standard Worker & Entity Retrieval Patterns

When developing integrations that retrieve worker or entity data, use the standard patterns below:

### 5.1 Active Contingent Worker Retrieval
- **Workday UI:** `Find Contingent Workers` with `Worker Status = Active`.
- **REST API:**
  `GET /ccx/api/staffing/v1/{tenant_name}/workers?workerType=Contingent_Worker&limit=1`
- **SOAP API (`Human_Resources`):**
  `Get_Workers` with `Include_Contingent_Workers_Only=true`, `Exclude_Inactive_Workers=true`, `Page=1`, `Count=1`.
- **RaaS Report:** Filter on `Contingent Worker` and `Active Status` with a 1-row limit.
- **Key Identifiers:** `Contingent_Worker_ID` and `WID`.

### 5.2 Active Regular Employee Retrieval
- **Workday UI:** `Find Workers` / `Find Employees` with `Worker Type = Employee` and `Worker Status = Active`.
- **REST API:**
  `GET /ccx/api/staffing/v1/{tenant_name}/workers?workerType=Employee&limit=1`
- **SOAP API (`Human_Resources`):**
  `Get_Workers` with `Include_Employees_Only=true`, `Exclude_Inactive_Workers=true`, `Page=1`, `Count=1`.
- **RaaS Report:** `All Active Employees` report filtered to `Employee` and `Active`.
- **Key Identifiers:** `Employee_ID` and `WID`.

### 5.3 SOAP Reference Extraction
Extract worker references from the response payload using specific `wd:type` attributes:
```xml
<!-- Extract WID -->
<xsl:value-of select="wd:Worker_Reference/wd:ID[@wd:type='WID']"/>

<!-- Extract Employee ID -->
<xsl:value-of select="wd:Worker_Reference/wd:ID[@wd:type='Employee_ID']"/>

<!-- Extract Contingent Worker ID -->
<xsl:value-of select="wd:Worker_Reference/wd:ID[@wd:type='Contingent_Worker_ID']"/>
```

---

## 6. Critical Lessons Learned & Troubleshooting Runbook

### 6.1 The `<redacted URL>` Namespace Corruption
* **Root Cause:** Web gateways, chat tools, or download systems can sanitize URL strings in XML payloads, converting `http://www.capeclear.com/assembly/10` into `<redacted URL>`. This corrupts the XML declaration and breaks Eclipse SAX parsing.
* **Remediation Rule:** Always use local in-place PowerShell scripts to generate and repair `assembly.xml` and `assembly-diagram.xml`. Use byte-array or character concatenation (e.g., `[string][char]47` for `/`) to construct namespace URIs safely.

### 6.2 Encoding & BOM Governance
* **Strict Rule:** Workday Studio XML files must be encoded in **UTF-8 without Byte Order Mark (BOM)**.
* In PowerShell, never use standard `Out-File -Encoding utf8` (which writes a BOM in Windows PowerShell 5.1). Always use:
  ```powershell
  $utf8NoBom = New-Object System.Text.UTF8Encoding($false)
  [System.IO.File]::WriteAllText($path, $content, $utf8NoBom)
  ```

### 6.3 PowerShell Command & Execution Safety
1. **Download Executable Shadowing:** If a downloaded `powershell.exe` exists in `Downloads`, running `powershell.exe` in Command Prompt executes the local binary instead of the Windows system binary. Always invoke:
   ```cmd
   "%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe"
   ```
2. **Process-Scoped Execution Policy:** Never permanently alter machine policies. Run scripts with temporary, process-scoped permission:
   ```cmd
   "%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -ExecutionPolicy RemoteSigned -File ".\script.ps1"
   ```
3. **Variable Parsing Colons:** Avoid expressions like `Write-Host "$path:"` which PowerShell interprets as drive prefixes (`$path:`). Use `$($path):` instead.

### 6.4 Studio Workspace Synchronization Protocol
Whenever `assembly.xml` or `assembly-diagram.xml` is modified on disk:
1. Close all open editor tabs in Workday Studio.
2. Select the project in **Project Explorer** and press **F5** (Refresh).
3. From the menu bar, run **Project → Clean...** on the target project.
4. Double-click `assembly.xml` to re-open the Graphical Assembly Editor.

---

## 7. Standard In-Place Generation Template

When creating a new Studio project or fixing an existing one, use this standard PowerShell structure to maintain safety:

```powershell
# ==============================================================================
# Workday Studio Canonical Assembly & Diagram Generator Template
# ==============================================================================
[CmdletBinding()]
param(
    [string]$Directory = 'C:\Path\To\ws\WSAR-INF'
)

$ErrorActionPreference = 'Stop'
Set-StrictMode -Version 2.0

$Directory = (Resolve-Path -LiteralPath $Directory).ProviderPath
$assemblyPath = Join-Path $Directory 'assembly.xml'
$diagramPath  = Join-Path $Directory 'assembly-diagram.xml'

# 1. Automatic Timestamped Backup
$timestamp = Get-Date -Format 'yyyyMMdd-HHmmss'
$backupDir = Join-Path $env:LOCALAPPDATA ("WorkdayStudio-Backups\Studio-" + $timestamp)
New-Item -ItemType Directory -Path $backupDir -Force | Out-Null
if (Test-Path -LiteralPath $assemblyPath) { Copy-Item -LiteralPath $assemblyPath -Destination $backupDir -Force }
if (Test-Path -LiteralPath $diagramPath)  { Copy-Item -LiteralPath $diagramPath  -Destination $backupDir -Force }

# 2. Write Assembly & Diagram via UTF-8 Without BOM
$utf8NoBom = New-Object System.Text.UTF8Encoding($false)
[System.IO.File]::WriteAllText($assemblyPath, $assemblyContent.Trim(), $utf8NoBom)
[System.IO.File]::WriteAllText($diagramPath,  $diagramContent.Trim(),  $utf8NoBom)

# 3. Structural & Parsing Verification
$settings = New-Object System.Xml.XmlReaderSettings
$settings.DtdProcessing = [System.Xml.DtdProcessing]::Prohibit
foreach ($f in @($assemblyPath, $diagramPath)) {
    $reader = [System.Xml.XmlReader]::Create($f, $settings)
    try {
        $doc = New-Object System.Xml.XmlDocument
        $doc.Load($reader)
    } finally {
        $reader.Dispose()
    }
}
Write-Host "Studio files successfully generated and verified." -ForegroundColor Green
```

---
*End of Master Specification — Maintained by Yogarajan Venkataraman (YV)*
