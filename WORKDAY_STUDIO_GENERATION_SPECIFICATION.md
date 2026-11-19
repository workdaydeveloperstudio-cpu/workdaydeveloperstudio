# WORKDAY_STUDIO_GENERATION_SPECIFICATION.md
**The Definitive Architecture, Generation Engine & Zero-Error Validation Specification for Workday Studio (`assembly.xml`, `assembly-diagram.xml`, and XSLT)**

---

## 1. Executive Root-Cause Matrix & Historical Engineering Lessons

Every failure pattern observed across production runs (**INT149C**, **INT0016**, **INT0076**, **INT0047**, and **Ad-Hoc Bank integrations**) is codified below into strict prevention rules:

| Failure Symptom | Observed Scenario | Root Cause | Preventive Implementation Rule |
|---|---|---|---|
| **`scala.MatchError` / Studio Crash** | Studio Eclipse visual editor open | Diagram references phantom components not in `assembly.xml`, or has broken nested swimlane links. | **The Triad Rule (Section 3.1):** Every component ID must exist identically across `<visualProperties>`, `<connections>`, and `<swimlanes>`. |
| **`HTTP 404 Not Found URL=.../ccx/service/null?...`** | INT149C RaaS report call (`08:20` & `12:17` runs) | String in `getExtrapath('Alias')` had typos/casing mismatches against `<cloud:report-alias name="...">`. | String passed to `getExtrapath()` must match the `name` attribute in `<cloud:report-alias>` character-for-character. |
| **`HTTP 400 Invalid Parm <name>`** | INT149C Cancelled Delta (`Updated_From`) | RaaS URL query parameter name did not match the **Label for Prompt XML Alias** on the custom report Prompts tab. | Never invent prompt keys. Verify via **Related Actions → Web Service → View URLs** before calling. |
| **`HTTP 500 Task not authorized`** | INT149C RealTime report call | Integration Security User (ISU) lacked View/Get permission on the report's Data Source domain (e.g., *Business Process Reporting*). | Grant View (Report/Task) + Get (Integration) on data source domain to ISSG; activate pending security policy changes. |
| **ISU Missing in Report Transfer Ownership Picker** | INT149C custom report ownership transfer | ISU lacked View/Get access to data source or filter fields, or lacked the *Custom Report Creation* domain. | Check permissions via *Security Analysis for Securable Item and Account*, or keep original owner and share the report directly. |
| **Misleading "Completed Successfully" on Failure** | INT149C & INT0016 delta extract failure | Extract transport error was caught, but `props['Error.Count']` was never incremented, letting `Success-PIM` fire. | Error mediations must explicitly increment `props['Error.Count'] = props['Error.Count'] + 1` so `Error-PIM` fires. |
| **Calling Real-Time vs Delta Report Mismatch** | INT149C `_RT_YV` vs `__CD_YV` | Renaming reports pointed delta assembly to a Real-Time report expecting `workdayID` instead of date range parameters. | Match assembly flow to report design: Event triggers use `workdayID`; Delta batch schedules use date range aliases. |
| **URL Sanitization `<redacted URL>` Syntax Error** | Downloading XML artifacts from chat environments | Download pipeline sanitizes `<redacted URL>...` in `xmlns` attributes into `<redacted URL>`, breaking XML parsers. | Deliver XML content or deliver via copy-paste Python scripts that construct namespace URIs from string fragments at runtime. |
| **`cvc-enumeration-valid: 'string'`** | INT149C launch parameters | `<cloud:simple-type>string</cloud:simple-type>` used instead of supported schema enumeration. | Schema enumeration allows only: `boolean`, `date`, `datetime`, `number`, `text`. |
| **`cc:log` element cannot have character children** | INT149C mediation logging | Placing bare text inside `<cc:log>` triggers Eclipse XML validation errors. | Use `<cc:cloud-log>` for audit logging, or nest `<cc:message><cc:text>...</cc:text></cc:message>` in `<cc:log>`. |
| **Duplicate Service Names in Collection** | Multi-project Studio collections (`.clc`) | Both projects used identical `<cloud:report-service name="INT_RS">`, causing build collisions. | Suffix service names per project (e.g., `_RT` for real-time and `_Delta` for scheduled batch). |
| **Inconsistent Assembly Versions** | Collections containing multiple CLARs | Projects in same collection mixed versions (e.g. `2025.21` and `2022.10`), leading to classloader errors. | Set matching versions across all assembly descriptors (e.g. `version="2022.10"`). |
| **QName Validation Error in XSLT** | Ad-Hoc Bank Transaction integration maps | Declaring `<xsl:param name="inmap.Bank Account"/>` containing spaces violates XML QName rules. | Never declare parameters with spaces. Retrieve mapped values using `is:getIntegrationMapValue('Map Name', value)`. |
| **Maximo Inbound 401 Unauthorized** | INT0076 Workorder Inbound after refresh | Tenant refresh left ISU authentication policy unactivated; username lacked tenant qualifier. | Ensure ISSG is in External ISU Access rule; activate auth policy changes; pass `user@tenant` format. |
| **Maximo BMXAA4137E on PO Status Change** | INT0047 Work Order PO update | PO had `STATUS=CLOSE` and `HISTORYFLAG=1`, so Maximo rejected updates to a closed record. | Add guards to suppress line numbers when PO is closed or check custom field aliases in raw CR XML before sending SOAP payload. |

---

## 2. `assembly.xml` Blueprint & Structural Template

### 2.1 Standard Header & Enclosing Beans
Every `assembly.xml` must open with standard Spring/Workday namespaces, followed by `<cc:assembly>`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
     xmlns="<redacted URL>"
     xmlns:beans="<redacted URL>"
     xmlns:atom="<redacted URL>"
     xmlns:cc="<redacted URL>"
     xmlns:cloud="urn:com.workday/esb/cloud/10.0"
     xmlns:env="<redacted URL>"
     xmlns:pi="urn:com.workday/picof"
     xmlns:wd="urn:com.workday/bsvc"
     xmlns:xsi="<redacted URL>">
 
    <cc:assembly id="WorkdayAssembly" version="2022.10">
        <!-- Components Go Here -->
    </cc:assembly>
</beans>
```

### 2.2 Integration Services & Allowed Schema Types
Declare integration services in `<cc:integration-system>` within `<cc:workday-in>`. Only use the allowed simple types: `boolean`, `date`, `datetime`, `number`, `text`.

```xml
<cc:workday-in id="StartHere" routes-to="Initialize">
    <cc:integration-system name="INT016 TBX Benefit Deductions Inbound">
        <!-- Launch Parameter Example -->
        <cloud:param name="Validation Mode">
            <cloud:type>
                <cloud:simple-type>boolean</cloud:simple-type>
            </cloud:type>
            <cloud:default>
                <cloud:boolean>true</cloud:boolean>
            </cloud:default>
        </cloud:param>

        <!-- Attribute Map Service -->
        <cloud:attribute-map-service name="INT016-Attribute-Map">
            <cloud:attribute name="Workday Web Service Version">
                <cloud:type>
                    <cloud:simple-type>text</cloud:simple-type>
                </cloud:type>
            </cloud:attribute>
            <cloud:attribute name="Document Retention Days">
                <cloud:type>
                    <cloud:simple-type>number</cloud:simple-type>
                </cloud:type>
            </cloud:attribute>
            <cloud:map name="Deduction">
                <cloud:internal-type>
                    <cloud:class-report-field description="Deductions" type="WID">d4a4b0e3f2334f999cd72662aa6f1b65</cloud:class-report-field>
                </cloud:internal-type>
                <cloud:external-type>
                    <cloud:simple-type>text</cloud:simple-type>
                </cloud:external-type>
            </cloud:map>
        </cloud:attribute-map-service>

        <!-- Report Service (Never hardcode RaaS URLs) -->
        <cloud:report-service name="INT016_RS">
            <cloud:report-alias description="INT016_Worker_Extract" name="INT016_Worker_Extract">
                <cloud:report-reference description="INT016_Worker_Extract" type="WID">ed5417169f9c10013fabd4ee26850000</cloud:report-reference>
            </cloud:report-alias>
        </cloud:report-service>

        <!-- Sequence Generator, Retrieval & Field Overrides -->
        <cloud:sequence-generator-service name="INT016-Batch">
            <cloud:sequencer>INT016-BatchID</cloud:sequencer>
        </cloud:sequence-generator-service>
        <cloud:retrieval-service name="INT016-Retrieval"/>
        <cloud:service-reference name="INT016-FieldOverride"/>
    </cc:integration-system>
</cc:workday-in>
```

### 2.3 Safe RaaS URL Resolution Pattern in Groovy
Always validate that `getExtrapath` does not return `null` before transport execution:

```groovy
def alias = 'INT016_Worker_Extract'
def basePath = intsys.reportService.getExtrapath(alias)
if (basePath == null) {
    throw new RuntimeException("Report alias '${alias}' returned null. Check cloud:report-service alias name.")
}
props['p.report.param'] = '?Supplier_Invoice_ID=' + java.net.URLEncoder.encode(props['pInvoiceRefID'].toString().trim(), 'UTF-8')
props['p.report.url'] = basePath + props['p.report.param']
```

### 2.4 Error Counting & Gated Completion Messaging Pattern
To ensure runs with failed steps never report success:

```xml
<!-- Error Handler: Log message and increment count -->
<cc:local-out id="ReportError-Out" routes-response-to="IncrementErrorCount" endpoint="vm://wcc/PutIntegrationMessage">
    <cc:set name="is.message.severity" value="'ERROR'"/>
    <cc:set name="is.message.summary" value="'Error invoking report: ' + context.errorMessage"/>
</cc:local-out>
<cc:async-mediation id="IncrementErrorCount">
    <cc:steps>
        <cc:eval id="Eval">
            <cc:expression>props['Error.Count'] = props['Error.Count'] + 1</cc:expression>
        </cc:eval>
    </cc:steps>
</cc:async-mediation>

<!-- Global Error Handler -->
<cc:send-error id="global-error-handler" rethrow-error="false" routes-to="Note-Error"/>
<cc:local-out id="Note-Error" endpoint="vm://wcc/PutIntegrationMessage">
    <cc:set name="is.message.severity" value="'CRITICAL'"/>
    <cc:set name="is.message.summary" value="'Fatal error at ' + context.errorComponentId"/>
    <cc:set name="is.message.detail" value="context.errorMessage"/>
</cc:local-out>

<!-- Gated Completion Messages -->
<cc:local-out id="Success-PIM" execute-when="props['Error.Count'] == 0" routes-response-to="Error-PIM" endpoint="vm://wcc/PutIntegrationMessage">
    <cc:set name="is.message.severity" value="'INFO'"/>
    <cc:set name="is.message.summary" value="'Integration completed successfully.'"/>
</cc:local-out>
<cc:local-out id="Error-PIM" execute-when="props['Error.Count'] > 0" endpoint="vm://wcc/PutIntegrationMessage">
    <cc:set name="is.message.severity" value="'ERROR'"/>
    <cc:set name="is.message.summary" value="'Integration completed with errors. Check log for details.'"/>
</cc:local-out>
```

---

## 3. `assembly-diagram.xml` Layout & Alignment Engine

To match the structure of `INT0016_assembly-diagram.xml` and `sample_assembly-diagram.xml`, follow these rules:

### 3.1 The Triad Rule (Eliminates `scala.MatchError`)
Every single executable component declared in `assembly.xml` with `id="MyComponent"` must appear in exactly three sections in `assembly-diagram.xml`:
1. `<visualProperties x="..." y="...">`: Declares position.
2. `<connections type="...">`: Declares inbound/outbound arrows.
3. `<swimlanes>`: Listed in `<elements href="assembly.xml#MyComponent"/>`.

### 3.2 Diagram Root Element & Global Error Handler Mapping
The root must reference the notation schema, and the global error handler must use EMF indexing:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wdnm:Diagram xmlns:wdnm="<redacted URL>">
  <element href="assembly.xml#WorkdayAssembly"/>

  <!-- CORRECT Global Error Handler Reference -->
  <visualProperties x="82" y="2090">
    <element href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
  </visualProperties>
  
  <connections type="routesTo">
    <source href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
    <target href="assembly.xml#Note-Error"/>
  </connections>
```
*(Do not use `href="assembly.xml#global-error-handler"`. Studio’s EMF generator indexes top-level unnamed beans by mixed position. Using the ID string causes an unresolved proxy crash).*

### 3.3 Visual Hierarchy & Coordinate Formulas
Structure diagrams with horizontal functional phases and nested vertical sublanes:

```
+------------------------------------------------------------------------------------------------------------------------+
| Swimlane 2: "INT016-Main" (Horizontal, Y=9, Font: Arial|14|1, FontColor: 1, LabelAlignment: LEFT)                     |
| [StartHere] -> [Initialize] (in Sublane 3) -> [GetIntMap] -> [CallFileRetrieval] -> [CallGetWorker] -> [PutMessage]    |
+------------------------------------------------------------------------------------------------------------------------+
| Swimlane 4: "FileRetrieval" (Horizontal, Y=354)                                                                        |
| [FileRetrieval] -> [Sublane 5: GetInputFile / Err] -> [Sublane 6: ValidateEmptyFile / EmptyFile]                       |
+------------------------------------------------------------------------------------------------------------------------+
| Swimlane 7: "GetWorker" (Horizontal, Y=354, X=743)                                                                     |
| [GetWorker] -> [Sublane 8: PrepWorkerWS / Err] -> [PagedGet] -> [GetPeriod]                                            |
+------------------------------------------------------------------------------------------------------------------------+
| Swimlane 9: "ProcessFile" (Horizontal, Y=749)                                                                          |
| [ProcessFile] -> [Sublane 10: MasterData] -> [Route0] -> [Sublane 11: PrepPayWS] -> [Splitter] -> [FinalizeWS] ...    |
+------------------------------------------------------------------------------------------------------------------------+
| Swimlane 17: "INT016-Audit" (Horizontal, Y=1348, TopBorderColor: 4227072)                                              |
| [INT016-Audit] -> [Sublane 16: Init-Audit] -> [CallRetrieveAudit] -> [CallGetImportMessage] -> [CallCloseAggAudit]     |
+------------------------------------------------------------------------------------------------------------------------+
| Swimlane 21: "RetrieveAudit" (Horizontal, Y=1687, TopBorderColor: 4227072)                                             |
| [RetriveAudit] -> [GetEventDocuments] -> [Sublane 19: AuditExist] -> [Route] -> [Sublane 20: UnzipFile]                |
+------------------------------------------------------------------------------------------------------------------------+
| Top-Right / Dedicated Error Swimlanes:                                                                                 |
| - "Critical Error Handler": X=1610, Y=142 -> [CriticalErr] -> [CriticalErrMsg] -> [ContinueFlag]                       |
| - "Global Error Handler":   X=1610, Y=21  -> [//@beans/@mixed.1/@mixed.39] -> [Note-Error]                             |
+------------------------------------------------------------------------------------------------------------------------+
```

### 3.4 Color, Font, and TopBorderColor Schemes
Use the verified integer color codes extracted from production diagrams:

| Visual Purpose | Color Type | Integer Code | Hex Equivalent | Description |
|---|---|---|---|---|
| **Main Process Boundary** | `fgColor` / `topBorderColor` | `65280` | `#00FF00` | Bright Green (Main Flow) |
| **Extraction / Processing**| `fgColor` / `topBorderColor` | `4194432` | `#400080` | Navy / Indigo (Core Data Extraction) |
| **WWS / Validation Calls** | `fgColor` / `topBorderColor` | `5942701` | `#5AA9AD` | Teal / Purple Slate (Web Service Operations) |
| **Audit & Reconcile Flow** | `topBorderColor` | `4227072` | `#408000` | Olive / Forest Green (Audit Phases) |
| **Vertical Sublanes** | `topBorderColor` | `33023` | `#0080FF` | Light Blue / Orange Highlight |
| **Global Error Handler** | `fgColor` / `topBorderColor` | `255` | `#0000FF` | Red / Blue Pure Alert |
| **Main Container Frame** | `fgColor` / `topBorderColor` | `16711680`| `#FF0000` | Studio Outer Boundary |

**Typography Standard:**
- Phase Headers: `font="Arial|14|1"` (Bold 14pt) with `fontColor="1"` (White) and `labelAlignment="LEFT"`
- Group Subheaders: `font="Arial|11|0"` (Regular 11pt)
- Step / Sublane Headers: `font="Arial|9|0"` (Regular 9pt)

### 3.5 Spacing and Coordinate Formulas
To avoid overlapping boxes or tangled connector lines:
1. **Horizontal Step Spacing ($\Delta X$):** 180px to 220px between sequential components in a row.
2. **Vertical Drop for Sublanes ($\Delta Y$):** 110px to 140px drop between a step and its error-call component.
3. **Inter-Group Vertical Margin:** Maintain at least 150px to 200px between horizontal functional bands.
4. **Sublane Orientation:** Always specify `orientation="VERTICAL"` when grouping a processing step with its corresponding `<cc:local-out>` error handler.

---

## 4. XSLT Standards & Workday Functions

### 4.1 Header, Output & Extension Prefixes
Use XSLT 2.0 and exclude extension prefixes to maintain clean XML outputs:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="2.0"
    xmlns:xsl="<redacted URL>"
    xmlns:xs="<redacted URL>"
    xmlns:wd="urn:com.workday/bsvc"
    xmlns:env="<redacted URL>"
    xmlns:is="java:com.workday.esb.intsys.xpath.ParsedIntegrationSystemFunctions"
    exclude-result-prefixes="xs is">

    <xsl:output method="xml" indent="yes" encoding="UTF-8"/>
```

### 4.2 Integration Map Value Lookups
Do not declare `<xsl:param name="inmap.Map Name"/>` when map names contain spaces. Use `is:getIntegrationMapValue()` inline:

```xml
<xsl:variable name="rawCompany" select="string(wd:Company_Code)"/>
<xsl:variable name="mappedCompany" select="is:getIntegrationMapValue('Company Map', $rawCompany)"/>

<wd:Company_Reference>
    <wd:ID wd:type="Company_Reference_ID">
        <xsl:value-of select="if ($mappedCompany != '') then $mappedCompany else $rawCompany"/>
    </wd:ID>
</wd:Company_Reference>
```

### 4.3 Safe Whitespace Stripping for Reference IDs
Prevent whitespace or line break formatting from polluting Workday ID values:

```xml
<!-- WRONG: Emits unwanted whitespace/newlines -->
<wd:ID wd:type="Tax_Code_ID">
    <xsl:value-of select="wd:Tax_Code"/>
</wd:ID>

<!-- CORRECT: Normalized inline -->
<wd:ID wd:type="Tax_Code_ID"><xsl:value-of select="normalize-space(wd:Tax_Code)"/></wd:ID>
```

### 4.4 Financial Formatting & Three-Tier Fallback Pattern
For tax reversals, adjustments, and amounts, implement cascading fallback logic:

```xml
<!-- Three-Tier Amount Extraction -->
<xsl:variable name="appliedAmount" select="
    if (number(wd:Down_Payment_Applied_Amount) > 0) then number(wd:Down_Payment_Applied_Amount)
    else if (number(wd:Total_Down_Payment_Applied) > 0) then number(wd:Total_Down_Payment_Applied)
    else number(substring-after(wd:Descriptor, '₹'))"/>

<!-- Absolute Formatting with Negative Sign for Reversals -->
<wd:Withholding_Tax_Amount>
    <xsl:value-of select="format-number(-1 * abs($appliedAmount * 0.01), '0.00')"/>
</wd:Withholding_Tax_Amount>
```

---

## 5. Automated Generation Workflow (Requirements to Artifacts)

When given business requirements, execute this 4-step sequence:

```
[Requirement Input]
        │
        ▼
1. Classify Architecture Type
   ├─ Real-Time Event Driven (Action Event prompt, 7 functional rows, pre-fetches)
   └─ Scheduled Delta Batch (Date range prompt, 3 functional rows, file retrieval/iterator)
        │
        ▼
2. Synthesize `assembly.xml`
   ├─ Set version="2022.10", declare services in <cc:integration-system>
   ├─ Use allowed simple-types: boolean, date, datetime, number, text
   ├─ Apply Safe RaaS resolution pattern in Groovy (test getExtrapath for null)
   ├─ Implement Error.Count incrementing on all error handlers
   └─ Gate Success-PIM with Error.Count == 0 and Error-PIM with Error.Count > 0
        │
        ▼
3. Synthesize `assembly-diagram.xml`
   ├─ Apply Triad Rule: Every bean ID must exist in visualProperties, connections, swimlanes
   ├─ Compute coordinates using Section 3.5 formulas (prevent overlapping)
   ├─ Enclose step + error pairs in vertical sublanes (orientation="VERTICAL")
   └─ Map global error handler via mixed index: assembly.xml#//@beans/@mixed.1/@mixed.X
        │
        ▼
4. Synthesize `*.xsl` Stylesheets
   ├─ Bind namespaces (wd, env, is) and set exclude-result-prefixes="xs is"
   ├─ Look up maps via is:getIntegrationMapValue('Name', val) (no param with space)
   └─ Wrap reference IDs in normalize-space()
```

---

## 6. Pre-Flight Verification Checklist

Before deploying any integration CLAR to a Workday tenant:

- [ ] **XML Namespaces:** No `<redacted URL>`, `<`, or markdown link syntax in `assembly.xml` or `assembly-diagram.xml`.
- [ ] **The Triad Match:** Every ID in `assembly.xml` exists in `assembly-diagram.xml` (`visualProperties`, `connections`, `swimlanes`).
- [ ] **Global Error Handler:** Diagram references `//@beans/@mixed.1/@mixed.X`, not `#global-error-handler`.
- [ ] **Report Service Aliases:** The string in `intsys.reportService.getExtrapath('...')` matches `<cloud:report-alias name="...">` verbatim.
- [ ] **Prompt XML Aliases:** Query parameter keys in RaaS URLs match the report's live **View URLs** query string.
- [ ] **Simple-Type Enumeration:** Launch parameters use only `boolean`, `date`, `datetime`, `number`, or `text`.
- [ ] **Unique Service Names:** In collections (`.clc`), report and attribute map services have project-unique names (e.g. `_RT`, `_Delta`).
- [ ] **Error Counter Logic:** Failed mediations increment `props['Error.Count']` so `Success-PIM` is bypassed on errors.
- [ ] **Tenant Security Policies:** Target ISU has View (Report/Task) + Get (Integration) on data source domains, and pending security changes have been activated.
- [ ] **Studio Clean Build:** Ran **Project → Clean...** in Eclipse with 0 errors in the Problems view.
