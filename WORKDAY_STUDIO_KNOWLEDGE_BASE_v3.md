# Workday Studio Architecture, Assembly & Diagram Master Knowledge Base (v3.0)

> **MANDATORY DIRECTIVE FOR ALL FUTURE ROVO WORKDAY STUDIO GENERATIONS:**  
> When creating, updating, or synchronizing `assembly.xml` and `assembly-diagram.xml`, **ALWAYS refer to this Knowledge Base contract**. Every rule herein represents an established, permanent engineering requirement. Violating any rule will cause Eclipse compilation errors, EMF deserialization crashes (`scala.MatchError`), or canvas layout overlaps.

---

## 1. Core Principles & Non-Negotiable Contract

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 PERMANENT WORKDAY STUDIO CONTRACT                                │
├─────────────────────────┬──────────────────────────┬─────────────────────────────────────────────┤
│ 1. Valid Namespace URIs │ 2. Zero Phantom Proxies  │ 3. Global Error Handler EMF Fragment        │
│ Never emit placeholder  │ Never reference an ID in │ Never reference top-level cc:send-error by  │
│ "<redacted URL>". Always│ assembly-diagram.xml that│ raw ID (#global-error-handler). Always use: │
│ use canonical schema    │ does not exist in        │ assembly.xml#//@beans/@mixed.1/@mixed.39    │
│ namespace URIs.         │ assembly.xml.            │ (Causes fatal scala.MatchError if violated) │
├─────────────────────────┼──────────────────────────┼─────────────────────────────────────────────┤
│ 4. Dedicated Rows       │ 5. Inter-Group Spacing   │ 6. Vertical Error Drops (No Overlap)        │
│ Never place two groups  │ Maintain >= 140px-160px  │ 5-step tall mediations require 220px-240px  │
│ side-by-side on same Y. │ clear vertical margin    │ vertical drop to error handlers. Simple     │
│ Each gets an exclusive  │ between preceding error  │ mediations require 170px-190px drop.        │
│ horizontal swimlane row.│ handlers and next group. │                                             │
├─────────────────────────┼──────────────────────────┼─────────────────────────────────────────────┤
│ 7. Canonical Names      │ 8. Three-Tier Fallback   │ 9. Delta Suppression Rule                   │
│ Never overwrite original│ Currency/Amount fallback:│ Cancelled delta integrations ONLY cancel    │
│ group header names. Keep│ Descriptor -> Applied -> │ prior tax invoices; they NEVER create or    │
│ exact naming conventions│ Total. Never crash on non│ submit replacement tax invoices.            │
│ across all diagrams.    │ INR currency symbols.    │                                             │
└─────────────────────────┴──────────────────────────┴─────────────────────────────────────────────┘
```

---

## 2. Namespace URIs Standard (`assembly.xml` Root Header)

### The "Missing URL / `<redacted URL>`" Defect
When exporting or generating `assembly.xml`, **never** allow XML namespace attributes to contain sanitized placeholder strings like `xmlns="<redacted URL>"`. Eclipse Studio's XML validator requires syntactically valid URI strings. If a placeholder is present, Eclipse marks line 3 with a red compilation error and fails project build.

### Mandatory Root Declaration
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
```

---

## 3. Eclipse EMF / GEF Architecture & `scala.MatchError` Elimination

### Technical Root Cause
Workday Studio uses Eclipse Graphical Editing Framework (GEF) and Eclipse Modeling Framework (EMF). In `Factory.scala` (`com.workday.sh.sa.editor.assembly.impl.editparts.Factory.createEditPart`), a Scala pattern-matching function inspects incoming model objects:
```scala
def createEditPart(context: EditPart, model: Any): EditPart = model match {
  case s: Swimlane => new SwimlaneEditPart(s)
  case m: Mediation => new MediationEditPart(m)
  case se: SendError => new SendErrorEditPart(se)
  // No case exists for org.eclipse.emf.ecore.impl.EObjectImpl!
}
```

When an element cannot be resolved, EMF creates an unmodeled proxy object of class `org.eclipse.emf.ecore.impl.EObjectImpl`. When GEF attempts to create an EditPart for it, Scala throws:
```
scala.MatchError: org.eclipse.emf.ecore.impl.EObjectImpl@... (of class org.eclipse.emf.ecore.impl.EObjectImpl)
    at com.workday.sh.sa.editor.assembly.impl.editparts.Factory.createEditPart(Factory.scala:19)
    at org.eclipse.gef.editparts.AbstractEditPart.createChild(AbstractEditPart.java:273)
```

### The Two Causes and Solutions

#### Cause A: Unresolved Proxy from Phantom IDs
* **Problem**: `assembly-diagram.xml` references `<element href="assembly.xml#NodeName"/>`, but `NodeName` does not exist in `assembly.xml` (e.g. `ResolveEvent`, `CallEventLookup`, `ProcessEventResponse`, or obsolete `Call-ExtractDownPayments_dnu`).
* **Rule**: **Zero Unresolved Proxies**. Every single `href="assembly.xml#ID"` in `assembly-diagram.xml` must map to an existing, declared element in `assembly.xml`.

#### Cause B: Top-Level `<cc:send-error>` Target
* **Problem**: `<cc:send-error id="global-error-handler">` is a top-level child of `<cc:assembly>`. In Cape Clear EMF schema, top-level `cc:send-error` tags do not index their `id` attribute into the EMF ID symbol table. Referencing `#global-error-handler` fails to resolve by ID and produces an `EObjectImpl`.
* **Rule**: The Global Error Handler **must ALWAYS** be referenced using the EMF FeatureMap XPath fragment:
  ```xml
  <visualProperties x="1610" y="21">
    <element href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
  </visualProperties>
  ```
  ```xml
  <connections type="routesTo" sourceAnchorX="0.9669" sourceAnchorY="0.4324" targetAnchorX="0.0606" targetAnchorY="0.3784">
    <source href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
    <target href="assembly.xml#Note-Error"/>
  </connections>
  ```
  ```xml
  <swimlanes x="1610" y="21" name="Global Error Handler" topBorderColor="255">
    <elements href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
    <elements href="assembly.xml#Note-Error"/>
  </swimlanes>
  ```

---

## 4. Visual Layout, Spacing & De-Overlapping Standards

### A. Dedicated Horizontal Rows (No Side-by-Side Collisions)
* In Eclipse GEF, swimlane containers calculate horizontal boundaries across the canvas. Placing two functional groups side-by-side at the same Y-coordinate (e.g., `Extract Withhold Tax Report` and `Extract GST Rate Report` both at $y=350$) causes their bounding boxes, title bars, and borders to collide and overlap.
* **Standard**: Every functional group gets its own dedicated row:
  * **Row 1**: `INT149C-Main` ($Y = 30$)
  * **Row 2**: `Extract Withhold Tax Report` ($Y = 360$)
  * **Row 3**: `Extract GST Rate Report` ($Y = 730$)
  * **Row 4**: `Extract Supplier Invoices` ($Y = 1110$)
  * **Row 5**: `Check for Tax Invoice` ($Y = 1510$)
  * **Row 6**: `Extract Downpayments` ($Y = 1960$)
  * **Row 7**: `Processing and creating tax invoices` ($Y = 2390$)

### B. Inter-Group Spacing Margin ($\ge 140\text{px} - 160\text{px}$)
* Always maintain a minimum of **$140\text{px} - 160\text{px}$ clear vertical margin** between the lowest error handler in a group and the top header boundary of the subsequent group.

### C. Vertical Error Drop Offsets (Tall Box Compensation)
* Mediation components expand vertically based on step count. A 5-step mediation (e.g., `Cancel-Tax_Invoice-WWS` with `Eval`, `CloudLog`, `Log`, `Write`, and `SendError`) measures $\approx 200\text{px}$ high.
* A standard $100\text{px}$ drop causes the error handler to render directly on top of the parent mediation.
* **Standards**:
  * Simple 1-step mediations: $\Delta Y = 170\text{px} - 190\text{px}$
  * Multi-step mediations (e.g., `Cancel-Tax_Invoice-WWS`): $\Delta Y = 240\text{px}$ drop (`Y=1570` $\rightarrow$ `Y=1810` or `1830`)
  * Transformation mediations (e.g., `ProcessReversalStep`): $\Delta Y = 210\text{px}$ drop (`Y=2490` $\rightarrow$ `Y=2700`)

### D. Canonical Group Header Names
Never alter, abbreviate, or overwrite canonical group names:
1. `INT149C-Main`
2. `Extract Withhold Tax Report`
3. `Extract GST Rate Report`
4. `Extract Supplier Invoices`
5. `Check for Tax Invoice`
6. `Extract Downpayments`
7. `Processing and creating tax invoices`
8. `Global Error Handler`

### E. Group 4 / Extract Downpayments 3-Column Layout
* In `Extract Downpayments`, the routing outcomes (`CallProcessDownPaymentsReversals0`, `WorktagsError`, `AsyncMediation`) must be stacked cleanly in a vertical sub-swimlane (`Swimlane 18`) at $X=1480$ with $\Delta Y = 65\text{px}$.

---

## 5. Swimlane Nesting & Indexing Standards

```
Swimlane 0: Global Error Handler
Swimlane 1: INT149C-Main (Row 1)
  └── Sub-Swimlane 2: Initialization (Vertical)
Swimlane 3: Extract Withhold Tax Report (Row 2)
  └── Sub-Swimlane 4: Withhold Error (Vertical)
Swimlane 5: Extract GST Rate Report (Row 3)
  └── Sub-Swimlane 6: GST Error (Vertical)
Swimlane 7: Extract Supplier Invoices (Row 4)
  └── Sub-Swimlane 8: Call Error (Vertical)
  └── Sub-Swimlane 9: Empty/Result Error (Vertical)
  └── Sub-Swimlane 10: DP Line Error (Vertical)
Swimlane 11: Check for Tax Invoice (Row 5)
  └── Sub-Swimlane 12: Find Tax Invoice Error (Vertical)
  └── Sub-Swimlane 13: Empty Tax Invoice Error (Vertical)
  └── Sub-Swimlane 14: Cancel SOAP Error (Vertical)
Swimlane 15: Extract Downpayments (Row 6)
  └── Sub-Swimlane 16: Downpayment Detail Error (Vertical)
  └── Sub-Swimlane 17: Calculation Error (Vertical)
  └── Sub-Swimlane 18: Route Outcomes (Vertical, 3-Column)
Swimlane 19: Processing and creating tax invoices (Row 7)
  └── Sub-Swimlane 20: Submit SOAP Error (Vertical)
```

* Parent horizontal swimlanes reference nested vertical swimlanes via `<elements href="#//@swimlanes.N"/>`.
* Never wrap multiple horizontal groups inside an overarching parent swimlane.
* All child indices (`0` through `20`) must be re-indexed consecutively upon any addition or removal.

---

## 6. Business Logic & Assembly Coding Standards

### A. Three-Tier Amount & Currency Fallback
Never rely on a single currency glyph (such as `₹`):
```java
// 1. Primary: Parse from descriptor substring
props['Individual_Amount'] = parts[0].xpath("substring-after(wd:Report_Data/wd:Report_Entry/wd:Down_Payment_Applications[contains(@wd:Descriptor, '" + props['invoice.number'] + "')]/@wd:Descriptor, '₹')");

// 2. Secondary Fallback: Applied Amount field
if (props['Individual_Amount'] == empty || props['Individual_Amount'] == '') {
    props['Individual_Amount'] = parts[0].xpath("wd:Report_Data/wd:Report_Entry/wd:Down_Payment_Applied_Amount");
}

// 3. Tertiary Fallback: Total Down Payment Applied field
if (props['Individual_Amount'] == empty || props['Individual_Amount'] == '') {
    props['Individual_Amount'] = parts[0].xpath("wd:Report_Data/wd:Report_Entry/wd:Total_Down_Payment_Applied");
}

props['Down_Payment_applied'] = props['Individual_Amount'];
```

### B. Dynamic API Versioning
Never hardcode static version literals like `v44.1` in assembly or XSLT:
```java
props['apiversion'] = intsys.getAttribute('API Version');
if (props['apiversion'] == empty || props['apiversion'] == '') {
    props['apiversion'] = 'v45.0';
}
```
Pass `@props['apiversion']` to all SOAP endpoints and `<xsl:param name="apiversion"/>` to stylesheets.

### C. Cancelled Delta vs. Real-Time Processing Rule
| Feature | Real-Time (`INT149C_Real_Time`) | Cancelled Delta (`INT149C_Cancelled_Delta`) |
| :--- | :--- | :--- |
| **Trigger** | Business Process Step (`Action_Event_WID`) | Scheduled / Ad-Hoc Datetime Window |
| **Driver Filter** | Specific `Supplier_Invoice_ID` (Approved) | `Status = Cancelled` |
| **Old Tax Invoice**| Cancel via SOAP `Cancel_Supplier_Invoice` | Cancel via SOAP `Cancel_Supplier_Invoice` |
| **New Tax Invoice**| **Create replacement via `submitsupplierinv.xslt`** | **SUPPRESSED COMPLETELY (Never submit)** |
| **Rate Pre-fetch** | Active (WHT & GST) | Suppressed / Not Queried |

---

## 7. XSLT Transformation Standards (`submitsupplierinv.xslt`)

1. **Memo Key Exactness**: `<wd:Memo>` must strictly equal:
   $$\text{concat}(\$invoice.number, '\_', \$Downpayment\_Invoice, '\_part2')$$
   This exact string enables `INT149B_Find_Tax_Invoices` to locate and cancel prior adjustments.
2. **Negative Withholding Amount**:
   ```xml
   <wd:Withholding_Tax_Amount>
       <xsl:value-of select="format-number(-1 * abs($Down_Payment_Final), '0.00')"/>
   </wd:Withholding_Tax_Amount>
   ```
3. **Zero Extended Amount**: Tax-only adjustments must set `<wd:Extended_Amount>0.00</wd:Extended_Amount>`. All financial impact resides inside the `<wd:Withholding_Tax_Data>` block.

---

## 8. Pre-Compilation Verification Checklist

Before compiling or saving any Workday Studio artifact, verify:
* [ ] Does `assembly.xml` use valid canonical namespace URIs without `<redacted URL>`?
* [ ] Does every single `id` referenced in `assembly-diagram.xml` exist in `assembly.xml`?
* [ ] Is the Global Error Handler referenced via `assembly.xml#//@beans/@mixed.1/@mixed.39`?
* [ ] Are `Extract Withhold Tax Report` and `Extract GST Rate Report` on separate horizontal rows?
* [ ] Is there at least $140\text{px}$ clear vertical space between all swimlane groups?
* [ ] Do multi-step mediations have at least $220\text{px} - 240\text{px}$ drop to their error handlers?
* [ ] Are all original group header names preserved verbatim?
* [ ] Does `Swimlane 18` implement the 3-column vertical layout for Route outcomes?
* [ ] Is the three-tier currency/amount fallback present in `CopyReport2`?
* [ ] For Cancelled Delta, is `Submit_Supplier_Invoice_Request` completely severed?
