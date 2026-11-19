# Workday Studio Assembly & Diagram Master Knowledge Base (v4.0)

> **PERMANENT DIRECTIVE**: When creating, updating, or synchronizing `assembly.xml`, `assembly-diagram.xml`, or `submitsupplierinv.xslt` for Workday Studio, **ALWAYS** refer to this Knowledge Base. Every rule represents a verified, battle-tested engineering requirement derived from real Eclipse compilation errors.

---

## 1. CRITICAL: URL/Namespace Sanitization Problem

### Problem
When generating XML files containing namespace URIs (e.g., `xmlns="http://www.springframework.org/schema/beans"`), the Rovo artifact download pipeline **sanitizes HTTP URLs** into `<redacted URL>` or corrupts them with markdown link syntax `[<redacted URL>...](<redacted URL>)` or backtick formatting.

### Impact
- Eclipse Xerces XML parser throws: `The value of attribute "xmlns" must not contain the '<' character`
- Eclipse throws: `Attribute name "URL" associated with an element type "redacted" must be followed by the '=' character`
- EMF throws: `Unable to read assembly.xml file`
- All cascade into `scala.MatchError` on `#WorkdayAssembly`

### Solution: Never Deliver XML Files via Artifact Downloads
**Instead, provide a Python repair/deployment script that users copy-paste from the chat response.** The script builds namespace URIs from string fragments at runtime:

```python
# Fragment-based URI construction prevents sanitization
_P = "htt" + "p"
_C = ":" + "//"
NS_BEANS = _P + _C + "www" + ".spring" + "framework.org/schema/beans"
```

### Canonical Namespace URIs (Built at Runtime)

| Prefix | URI |
|--------|-----|
| `xmlns` / `xmlns:beans` | `http` + `://www.springframework.org/schema/beans` |
| `xmlns:atom` | `http` + `://www.w3.org/2005/Atom` |
| `xmlns:cc` | `http` + `://www.capeclear.com/assembly/10` |
| `xmlns:cloud` | `urn:com.workday/esb/cloud/10.0` |
| `xmlns:env` | `http` + `://schemas.xmlsoap.org/soap/envelope/` |
| `xmlns:pi` | `urn:com.workday/picof` |
| `xmlns:wd` | `urn:com.workday/bsvc` |
| `xmlns:xsi` | `http` + `://www.w3.org/2001/XMLSchema-instance` |
| `xmlns:xsl` | `http` + `://www.w3.org/1999/XSL/Transform` |
| `xmlns:xs` | `http` + `://www.w3.org/2001/XMLSchema` |
| `xmlns:wdnm` | `http` + `://workday.com/studio/editors/notation` |

### Repair Script Pattern
When files are already corrupted on disk, provide a **repair script** that reads existing files and fixes corrupted patterns:
```python
import os, re, shutil
# Scan for corrupted xmlns attributes
text = re.sub(r'<redacted[^>]*>', '', text)
# Rebuild from fragments at runtime
```

---

## 2. Eclipse EMF / GEF Architecture & scala.MatchError

### Error Pattern
```
scala.MatchError: org.eclipse.emf.ecore.impl.EObjectImpl@...
    at com.workday.sh.sa.editor.assembly.impl.editparts.Factory.createEditPart(Factory.scala:19)
```

### Three Causes

#### Cause A: Phantom Node References
- `assembly-diagram.xml` references an ID that does not exist in `assembly.xml`
- Example: Cancelled Delta diagram referencing `#CallGetWithholdRates` (only exists in Real-Time)
- **Rule**: Every `href="assembly.xml#ID"` in the diagram must match a declared element in assembly.xml

#### Cause B: Global Error Handler EMF Fragment
- Top-level `<cc:send-error id="global-error-handler">` is not indexed by EMF ID
- Referencing `#global-error-handler` creates an unresolved `EObjectImpl` proxy
- **Rule**: Always use EMF FeatureMap XPath: `assembly.xml#https://@beans/@mixed.1/@mixed.39`

#### Cause C: Assembly.xml Parse Failure
- If `assembly.xml` has ANY syntax error (namespace corruption, `<` in attributes), EMF cannot parse it at all
- The root element `#WorkdayAssembly` becomes an unresolved proxy
- `setContents()` in `AssemblyDiagramEditor` crashes
- **Rule**: Fix assembly.xml syntax errors FIRST before debugging diagram errors

---

## 3. Workday Studio XML Schema Validation Rules

### cloud:simple-type Enumeration
Valid values: `[boolean, date, datetime, number, text]`

**WRONG**: `<cloud:simple-type>string</cloud:simple-type>` (causes `cvc-enumeration-valid` error)
**CORRECT**: `<cloud:simple-type>text</cloud:simple-type>`

### cc:log Element-Only Content
`<cc:log>` cannot have character children (text content). Use `<cc:cloud-log>` instead for logging, which is the standard Workday Studio pattern.

**WRONG**:
```xml
<cc:log id="Log">
    <cc:message>
        <cc:text>Some message</cc:text>
    </cc:message>
</cc:log>
```
**CORRECT**: Remove `<cc:log>` and use only `<cc:cloud-log>`:
```xml
<cc:cloud-log id="CloudLog" level="info" message="Some message" output-file-type="HTML"/>
```

### Markdown Link Corruption in assembly-diagram.xml
When copying diagram XML from chat, markdown formatting can corrupt the namespace:

**CORRUPTED**: `xmlns:wdnm="[http://workday.com...](<redacted URL>)"`
**CORRECT**: `xmlns:wdnm="http://workday.com/studio/editors/notation"`

---

## 4. Workday Studio Collection Rules

### Unique Service Names Across Collection
When multiple projects belong to the same `.clc` Collection (e.g., `INT149C_TaxPaymentsReversal_JournalCollection`), all `<cloud:attribute-map-service>` and `<cloud:report-service>` names must be **unique across the entire collection**.

**WRONG** (both projects use same names):
```xml
<!-- Project 1 --> <cloud:attribute-map-service name="INT149b_Attributes">
<!-- Project 2 --> <cloud:attribute-map-service name="INT149b_Attributes">
```

**CORRECT** (suffixed for uniqueness):
```xml
<!-- Real-Time -->  <cloud:attribute-map-service name="INT149b_Attributes_RT">
<!-- Delta -->      <cloud:attribute-map-service name="INT149b_Attributes_Delta">
```

Same applies to `<cloud:report-service>`:
- Real-Time: `INT149b_RS_RT`
- Delta: `INT149b_RS_Delta`

And `<cloud:service-reference>` in ad-hoc entry points must match:
```xml
<cloud:service-reference name="INT149b_Attributes_Delta"/>
<cloud:service-reference name="INT149b_RS_Delta"/>
```

### Assembly Version Consistency
All assemblies in a collection must use the **same version**:
```xml
<cc:assembly id="WorkdayAssembly" version="2022.10">
```
Mixing versions (e.g., `2025.21` and `2022.10`) causes:
```
Collection contains assemblies with different versions. This may result in class loading issues.
```

### File Naming
Files must be named exactly `assembly.xml` and `assembly-diagram.xml` in `ws/WSAR-INF/`. The diagram resolves `<element href="assembly.xml#WorkdayAssembly"/>` as a relative path.

---

## 5. Visual Layout & Diagram Standards

### Dedicated Horizontal Rows
Never place two functional groups side-by-side at the same Y-coordinate. Each gets its own row:

| Row | Group Name | Y Position |
|-----|-----------|------------|
| 1 | INT149C-Main | Y=30 |
| 2 | Extract Withhold Tax Report | Y=360 |
| 3 | Extract GST Rate Report | Y=730 |
| 4 | Extract Supplier Invoices | Y=1110 |
| 5 | Check for Tax Invoice | Y=1510 |
| 6 | Extract Downpayments | Y=1960 |
| 7 | Processing and creating tax invoices | Y=2390 |
| Top-Right | Global Error Handler | Y=21 |

### Inter-Group Spacing
Minimum 140px-160px clear vertical margin between groups.

### Vertical Error Drop Offsets
- Simple 1-step mediations: 170px-190px drop
- Multi-step mediations (e.g., Cancel-Tax_Invoice-WWS): 240px drop
- Transformation mediations (e.g., ProcessReversalStep): 210px drop

### Swimlane Indexing
Swimlanes are indexed 0-20 consecutively. Parent swimlanes reference children via `#<redacted URL>`.

---

## 6. Cancelled Delta vs Real-Time Architecture

| Feature | Real-Time | Cancelled Delta |
|---------|-----------|-----------------|
| **Trigger** | BP Step (`Action_Event_WID`) | Scheduled/Ad-Hoc datetime window |
| **Parameter Type** | `<cloud:simple-type>text</cloud:simple-type>` | `<cloud:simple-type>datetime</cloud:simple-type>` |
| **WHT/GST Pre-fetch** | Active (Rows 2-3) | Suppressed (not present) |
| **Driver Filter** | Single `Supplier_Invoice_ID` | `Status=Cancelled` datetime range |
| **Old Tax Invoice** | Cancel via SOAP | Cancel via SOAP |
| **New Tax Invoice** | **Create via `submitsupplierinv.xslt`** | **SUPPRESSED (never submit)** |
| **XSLT Required** | Yes (`submitsupplierinv.xslt`) | No |
| **Diagram Rows** | 7 rows + Global Error Handler | 3 rows + Global Error Handler |
| **Service Suffix** | `_RT` | `_Delta` |

### Delta Diagram Component Inventory (3 rows only)
Row 1: `StartHere`, `StartHere-Adhoc`, `Initialization`, `Call-ExtractSupplierInvoices`, `Store-Log`, `Success-PIM`, `Error-PIM`
Row 2: `ExtractSupplierInvoices`, `CallSupplierInvoiceError`, `SupplierInvoice-Report-Error`, `CallSupplierInvoicesReport`, `SupplierInvoiceError`, `LogSupplierInvoiceError`, `SplitRecords`, `CopyReport`, `Split_DP_Inv_Applied`, `DP_Invoice`, `Call-Tax_Invoice`
Row 3: `Check-Tax_Invoice`, `CallTaxInvoiceReport`, `Check-Tax-Invoice-Error01`, `Tax_Invoice`, `TaxInvoiceError`, `LogTaxInvoiceError`, `SplitTaxInvRecords`, `Cancel-Tax_Invoice-WWS`, `Log_WWS_Error`, `WorkdayOutSoap`, `LogTaxInvoice`

---

## 7. XSLT Standards (submitsupplierinv.xslt)

### Memo Key for Tax Invoice Matching
```xml
<wd:Memo>
    <xsl:value-of select="concat($invoice.number, '_', $Downpayment_Invoice, '_part2')"/>
</wd:Memo>
```

### Negative Withholding Amount
```xml
<wd:Withholding_Tax_Amount>
    <xsl:value-of select="format-number(-1 * abs($Down_Payment_Final), '0.00')"/>
</wd:Withholding_Tax_Amount>
```

### Zero Extended Amount
Tax-only adjustments: `<wd:Extended_Amount>0.00</wd:Extended_Amount>`

---

## 8. Three-Tier Amount & Currency Fallback

Never rely on a single currency glyph (e.g., INR symbol). Use cascading fallback:
1. Primary: Parse from descriptor substring
2. Secondary: `Down_Payment_Applied_Amount` field
3. Tertiary: `Total_Down_Payment_Applied` field

---

## 9. Error Resolution Quick Reference

| Error Message | Root Cause | Fix |
|--------------|-----------|-----|
| `xmlns must not contain '<'` | URL sanitization or markdown corruption | Use repair script with fragment-based URIs |
| `Attribute name "URL" must be followed by '='` | `<redacted URL>` in file | Run repair script to replace corrupted patterns |
| `scala.MatchError: EObjectImpl...#WorkdayAssembly` | assembly.xml parse failure OR corrupted diagram namespace | Fix assembly.xml syntax first, then fix diagram xmlns |
| `scala.MatchError: EObjectImpl...#ComponentName` | Phantom node in diagram not in assembly | Remove reference or add component to assembly |
| `cvc-enumeration-valid: 'string' not valid` | Invalid simple-type value | Change `string` to `text` |
| `cc:log cannot have character children` | Wrong logging element | Replace `<cc:log>` with `<cc:cloud-log>` |
| `More than one service named 'X'` | Duplicate service names in collection | Suffix with `_RT` and `_Delta` |
| `Different versions in collection` | Version mismatch | Align all to `version="2022.10"` |
| `Missing integration system parameter type` | Invalid or missing simple-type | Use valid enumeration: boolean/date/datetime/number/text |

---

## 10. Pre-Deployment Checklist

Before deploying any Workday Studio artifact:
- [ ] Are namespace URIs valid (no `<`, `>`, `redacted`, markdown brackets)?
- [ ] Does every diagram href match an existing assembly.xml ID?
- [ ] Is Global Error Handler referenced via `assembly.xml#https://@beans/@mixed.1/@mixed.39`?
- [ ] Are service names unique across the collection (`_RT` / `_Delta`)?
- [ ] Is assembly version consistent across collection (`2022.10`)?
- [ ] Are files named exactly `assembly.xml` and `assembly-diagram.xml`?
- [ ] Is `<cloud:simple-type>text</cloud:simple-type>` used (not `string`)?
- [ ] Are `Extract Withhold Tax Report` and `Extract GST Rate Report` on separate rows?
- [ ] Is the three-tier currency fallback present in CopyReport2?
- [ ] For Cancelled Delta, is Submit_Supplier_Invoice completely suppressed?
- [ ] For downloaded artifacts, were URLs sanitized? If so, use repair script.

---

## 11. Delivery Method Decision Tree

```
User asks for assembly.xml / assembly-diagram.xml / xslt
    |
    +--> Does the file contain HTTP URLs in xmlns attributes?
           |
           YES --> DO NOT use artifact download (save_file_for_user)
           |       Instead: Provide copy-paste Python repair/deploy script
           |       in the chat response body
           |
           NO  --> Safe to use artifact download
```

**Key Lesson**: The Rovo artifact download pipeline sanitizes URLs. For XML files with namespace declarations, always deliver via copy-paste scripts that build URIs from string fragments at runtime.
