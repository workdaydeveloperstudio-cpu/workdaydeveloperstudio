# Workday Studio Architecture & Diagram Engineering Specification (v2.0 Latest)

---

## 1. Executive Summary & Permanent Memory Rules

This specification establishes the official, production-grade engineering rules for creating, synchronizing, and modifying Workday Studio `assembly.xml` and `assembly-diagram.xml` files. Adherence to these rules completely prevents Eclipse EMF/GEF deserialization crashes (e.g., `scala.MatchError`), eliminates visual component/swimlane overlaps, and ensures clean, maintainable integration systems.

---

## 2. Preventing `scala.MatchError` & Eclipse EMF Deserialization Crashes

### 2.1 The Underlying Root Cause
Workday Studio’s graphical editor is built upon Eclipse Graphical Editing Framework (GEF) and Eclipse Modeling Framework (EMF), with layout logic written in Scala (`AssemblyDiagramEditor.scala`, `Factory.scala`).
* When `assembly-diagram.xml` contains `<element href="assembly.xml#NodeID"/>`, EMF attempts to resolve `NodeID` against the assembly's Cape Clear XML schema definitions.
* If `NodeID` does not exist in `assembly.xml`, or if the XML element does not expose an EMF-indexed XML `ID` attribute, EMF fails to resolve the proxy and instantiates a generic `org.eclipse.emf.ecore.impl.EObjectImpl`.
* When `SwimlaneEditPart.refreshChildren` passes `EObjectImpl` to `Factory.createEditPart(model: Any)`, the Scala pattern-matching function in Studio's GEF factory has no case for an unresolved proxy and crashes immediately:
  ```
  scala.MatchError: org.eclipse.emf.ecore.impl.EObjectImpl@... (eProxyURI: platform:/resource/.../assembly.xml#NodeID)
      at com.workday.sh.sa.editor.assembly.impl.editparts.Factory.createEditPart(Factory.scala:19)
      at org.eclipse.gef.editparts.AbstractEditPart.createChild(AbstractEditPart.java:273)
  ```

### 2.2 The Two Immutable Rules to Eliminate `scala.MatchError`
1. **Zero Phantom Node References**:
   * **Rule**: Never reference an element ID in `assembly-diagram.xml` that is not declared and present in `assembly.xml`.
   * **Common Trap**: Retaining visual properties or swimlane elements for components that were refactored or deleted (e.g., referencing `ResolveEvent`, `CallEventLookup`, `ProcessEventResponse`, or `Call-ExtractDownPayments_dnu` when those nodes do not exist in the active `assembly.xml`).
2. **Top-Level Error Handler EMF Fragment Rule**:
   * **Rule**: Never reference top-level `<cc:send-error>` by a raw string ID (e.g., `assembly.xml#global-error-handler`).
   * **Reason**: In the Cape Clear assembly schema, top-level `cc:send-error` does not expose an EMF-indexed ID attribute. Referencing it by ID always returns an unresolved `EObjectImpl` proxy.
   * **Mandatory Solution**: Always reference the top-level error handler using its exact EMF FeatureMap XPath fragment:
     ```xml
     <visualProperties x="1610" y="21">
       <element href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
     </visualProperties>
     ...
     <swimlanes x="1610" y="21" name="Global Error Handler" topBorderColor="255">
       <elements href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
       <elements href="assembly.xml#Note-Error"/>
     </swimlanes>
     ```

---

## 3. Swimlane Architecture & Layout De-Overlapping Standards

### 3.1 Preservation of Exact Group Header Names
Do not rename or merge functional swimlane titles. The canonical names must be preserved verbatim:
1. `INT149C-Main`
2. `Extract Withhold Tax Report`
3. `Extract GST Rate Report`
4. `Extract Supplier Invoices`
5. `Check for Tax Invoice`
6. `Extract Downpayments`
7. `Processing and creating tax invoices`
8. `Global Error Handler`

### 3.2 Elimination of Horizontal Swimlane Overlap
* **The Defect**: Placing two horizontal swimlanes side-by-side at the same vertical offset ($Y$) causes their canvas boundaries, header bars, and borders to collide and overlap.
* **The Standard**: **Every functional process must reside on its own independent horizontal row**.
  * *Specific Example*: `Extract Withhold Tax Report` (Row 2, $Y=360$) and `Extract GST Rate Report` (Row 3, $Y=730$) must never share the same Y-axis level. They must be stacked in separate horizontal swimlanes.

### 3.3 Clear Vertical Line Spacing & Margin Rules
* **Inter-Group Line Spacing**: Maintain **$140\text{px} - 160\text{px}$ of clear, blank vertical space** between the bottom-most error handlers of one group and the starting header of the next group.
* **Intra-Group Vertical Drop for Error Handlers**:
  * Simple mediations: $170\text{px} - 190\text{px}$ vertical drop.
  * Multi-step mediations (e.g., `Cancel-Tax_Invoice-WWS` containing `Eval`, `CloudLog`, `Log`, `Write`, and `SendError`, reaching $\approx 200\text{px}$ box height): Require a **$220\text{px} - 240\text{px}$ vertical drop** to guarantee zero visual overlap with the error handler below.

### 3.4 Standard Coordinate Matrix (Reference Grid)

| Group / Row # | Swimlane Header Name | Swimlane $(X, Y)$ | Main Step $Y$ | Error Handler $Y$ | Vertical Gap Below |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Top-Right** | `Global Error Handler` | $(1610, 21)$ | $Y = 21$ | N/A | Dedicated Canvas Corner |
| **Row 1** | `INT149C-Main` | $(31, 30)$ | $Y = 80 - 110$ | $Y = 160$ (Adhoc/PIM) | $\approx 130\text{px}$ clear gap |
| **Row 2** | `Extract Withhold Tax Report` | $(31, 360)$ | $Y = 410 - 430$ | $Y = 580$ ($170\text{px}$ drop) | $\approx 150\text{px}$ clear gap |
| **Row 3** | `Extract GST Rate Report` | $(31, 730)$ | $Y = 780 - 800$ | $Y = 950$ ($170\text{px}$ drop) | $\approx 160\text{px}$ clear gap |
| **Row 4** | `Extract Supplier Invoices` | $(31, 1110)$ | $Y = 1170 - 1190$| $Y = 1360$ ($190\text{px}$ drop) | $\approx 150\text{px}$ clear gap |
| **Row 5** | `Check for Tax Invoice` | $(31, 1510)$ | $Y = 1570 - 1600$| $Y = 1810$ ($240\text{px}$ drop) | $\approx 150\text{px}$ clear gap |
| **Row 6** | `Extract Downpayments` | $(31, 1960)$ | $Y = 2030 - 2050$| $Y = 2230$ ($200\text{px}$ drop) | $\approx 160\text{px}$ clear gap |
| **Row 7** | `Processing and creating tax invoices`| $(31, 2390)$ | $Y = 2460 - 2480$| $Y = 2670$ ($210\text{px}$ drop) | End of canvas |

---

## 4. Swimlane Indexing & Nesting Rules

Workday Studio diagrams model nested error blocks using vertical sub-swimlanes.

1. **Top-Level Horizontal Swimlanes**: Define broad functional phases (`font="Arial|11|1"`, `fgColor`, `topBorderColor`).
2. **Nested Vertical Sub-Swimlanes**: Encapsulate mediation-error pairs (`orientation="VERTICAL"`).
3. **Index-Based Cross References**:
   * Parent horizontal swimlanes reference child vertical swimlanes via `<elements href="#//@swimlanes.N"/>`.
   * **Rule**: Swimlane indices are zero-based (`0` to $N$). If a swimlane is added or deleted, all child references must be re-indexed to prevent swimlane detachment.
   * **Rule**: Never create an outer "wrapper" swimlane enclosing other horizontal swimlanes (e.g., eliminating the legacy `INT148a Sagility Receipt Accural Boomerang Studio` master wrapper).

---

## 5. Flow & Connection Anchor Standards

* **Forward Process Flow (Left-to-Right)**:
  ```xml
  <connections type="routesTo" sourceAnchorX="1.0" sourceAnchorY="0.5" targetAnchorX="0.0" targetAnchorY="0.5">
  ```
* **Synchronous Local Sub-Flow Return (Left-to-Right)**:
  ```xml
  <connections type="routesResponseTo" sourceAnchorX="1.0" sourceAnchorY="0.5" targetAnchorX="0.0" targetAnchorY="0.5">
  ```
* **Downward Error Flow (Top-to-Bottom)**:
  ```xml
  <connections type="routesTo" sourceAnchorX="0.5" sourceAnchorY="1.0" targetAnchorX="0.5" targetAnchorY="0.0">
  ```

---

## 6. Workday Studio `assembly.xml` Engineering Standards

### 6.1 Launch Configuration Patterns
* **BP-Triggered Real-Time Pattern**:
  * Must accept a single required parameter: `Action_Event_WID` (string).
  * Requires pre-query lookup report (`INT149_Find_Supplier_Invoice_By_Event`) to resolve `Action_Event_WID` $\rightarrow$ `Supplier_Invoice_Reference_ID`.
  * Driver report queries by single invoice ID: `?Supplier_Invoice_ID=@{props['resolved.Invoice_Ref_ID']}`.
  * Omit `StartHere-Adhoc` to prevent uncoordinated manual launches without event context.
* **Scheduled / Ad-Hoc Delta Pattern**:
  * Accepts `Updated From` and `Updated To` datetime parameters.
  * Includes both `StartHere` (scheduled) and `StartHere-Adhoc` (manual).
  * Filter driver report by `Invoice Status = Cancelled` or `Status = Approved`.

### 6.2 Hardened Amount & Currency Parsing
Never use raw substring matching on a single localized currency symbol (e.g. `substring-after(..., '₹')`). Always implement a three-tier fallback:
```java
props['Individual_Amount'] = parts[0].xpath("substring-after(wd:Report_Data/wd:Report_Entry/wd:Down_Payment_Applications[contains(@wd:Descriptor, '" + props['invoice.number'] + "')]/@wd:Descriptor, '₹')");

if (props['Individual_Amount'] == empty || props['Individual_Amount'] == '') {
    props['Individual_Amount'] = parts[0].xpath("wd:Report_Data/wd:Report_Entry/wd:Down_Payment_Applied_Amount");
}
if (props['Individual_Amount'] == empty || props['Individual_Amount'] == '') {
    props['Individual_Amount'] = parts[0].xpath("wd:Report_Data/wd:Report_Entry/wd:Total_Down_Payment_Applied");
}
props['Down_Payment_applied'] = props['Individual_Amount'];
```

### 6.3 Dynamic API Versioning
* Never hardcode API versions in mediation logic or SOAP invocations.
* Configure an Integration Attribute Map:
  ```xml
  <cloud:attribute-map-service name="INT149b_Attributes">
      <cloud:attribute name="API Version">
          <cloud:type>
              <cloud:class-report-field description="Web Service API Version" type="WID" singular="true">643097491d2410000a9da38c99860026</cloud:class-report-field>
          </cloud:type>
      </cloud:attribute>
  </cloud:attribute-map-service>
  ```
* Dynamically resolve in `Initialization`:
  ```java
  props['apiversion'] = intsys.getAttribute('API Version');
  if (props['apiversion'] == empty || props['apiversion'] == '') {
      props['apiversion'] = 'v45.0';
  }
  ```
* Pass `version="@{props['apiversion']}"` to all `<cc:workday-out-soap>` components.

### 6.4 Cancellation Delta Reversal Suppression
On delta runs processing cancelled supplier invoices:
* Match and cancel existing tax-only invoices (`<invoice>_<DP>_part2`) via `Cancel_Supplier_Invoice_Request`.
* **Suppress all replacement tax invoice submissions**. Do not invoke `submitsupplierinv.xslt` or post replacement invoices on cancelled transactions.

---

## 7. Developer Pre-Deployment Verification Checklist

Before importing or compiling Workday Studio packages:
1. [ ] **Proxy Validation**: Perform text search in `assembly-diagram.xml` for `href="assembly.xml#..."`. Cross-reference every single target ID against `assembly.xml`. Verify 0 missing IDs.
2. [ ] **Global Error Check**: Ensure top-level error catcher is referenced via `href="assembly.xml#//@beans/@mixed.1/@mixed.39"`.
3. [ ] **Row Separation**: Verify `Extract Withhold Tax Report` and `Extract GST Rate Report` are on separate rows with distinct $Y$ coordinates.
4. [ ] **Gap Verification**: Confirm all horizontal groups have $\ge 140\text{px}$ vertical line spacing.
5. [ ] **Error Drop Clearance**: Confirm multi-step mediations have $\ge 220\text{px}$ vertical drop to their respective error handlers.
6. [ ] **Swimlane Indexing**: Validate all `#//@swimlanes.N` references accurately point to the intended vertical child lanes.
7. [ ] **Currency Robustness**: Verify amount extraction logic includes fallback XPath expressions for non-INR currencies.
