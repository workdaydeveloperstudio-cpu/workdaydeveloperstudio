# WORKDAY STUDIO COMPREHENSIVE KNOWLEDGE BASE: ASSEMBLY & ASSEMBLY-DIAGRAM STANDARDS

> **Document Version**: 2.0  
> **Target Audience**: Workday Studio Developers, Integration Architects, AI Assistants  
> **Purpose**: Serves as the authoritative blueprint and definitive design guide for generating, refactoring, and maintaining production-grade Workday Studio `assembly.xml` and `assembly-diagram.xml` files with **zero Eclipse EMF/GEF `scala.MatchError` crashes**, pixel-perfect visual layouts, and strict schema compliance.

---

## 1. Executive Rules & Core Principles

1. **Every Studio Component in `assembly-diagram.xml` Must Exist in `assembly.xml`**:
   - Never reference a component ID in `assembly-diagram.xml` that is not declared in `assembly.xml`. Doing so causes Eclipse EMF to instantiate an unresolved proxy (`EObjectImpl`), triggering an unhandled Scala pattern match exception: `scala.MatchError: org.eclipse.emf.ecore.impl.EObjectImpl`.
2. **Never Target Top-Level `<cc:send-error>` by Raw String ID**:
   - Top-level error catchers (such as `<cc:send-error id="global-error-handler">`) do not inherit from the standard EMF EObject ID registry in the Cape Clear metamodel.
   - Referencing `assembly.xml#global-error-handler` causes `Factory.createEditPart` to fail immediately.
   - **Mandatory Syntax**: Always reference top-level send-errors via their EMF XPath fragment:
     ```xml
     assembly.xml#//@beans/@mixed.1/@mixed.39
     ```
3. **Strict 3-Level Swimlane Hierarchy**:
   - Level 1: Canvas / Primary container.
   - Level 2: Functional horizontal process swimlanes (e.g., `INT-Main`, `Extract Data`, `Check Status`, `Process Records`).
   - Level 3: Nested vertical error-handling/sub-flow swimlanes (`orientation="VERTICAL"`), referenced by index (`<elements href="#//@swimlanes.N"/>`).
4. **No Giant Enclosing Outer Swimlanes**:
   - Never wrap multiple distinct functional horizontal swimlanes in a single master swimlane (such as legacy `INT148a Sagility Receipt Accural...` artifacts). Each functional step must be an independent horizontal row.
5. **Standardized Anchor Orientations**:
   - **Forward Sequential Routes** (`routesTo`, `routesResponseTo`): Left-to-Right (`sourceAnchorX="1.0" sourceAnchorY="0.5"` to `targetAnchorX="0.0" targetAnchorY="0.5"`).
   - **Downstream Error Routes** (`routesTo` from error steps): Top-to-Bottom (`sourceAnchorX="0.5" sourceAnchorY="1.0"` to `targetAnchorX="0.5" targetAnchorY="0.0"`).
   - **Branch Routing** (`<cc:route>`): Vertical distribution (`sourceAnchorX="1.0" sourceAnchorY="0.25"`, `0.50"`, `0.75"`).

---

## 2. Deep Dive: Eclipse EMF, GEF & `scala.MatchError` Prevention

### The Anatomy of the Crash
```
scala.MatchError: org.eclipse.emf.ecore.impl.EObjectImpl@... (eProxyURI: platform:/resource/.../assembly.xml#...)
    at com.workday.sh.sa.editor.assembly.impl.editparts.Factory.createEditPart(Factory.scala:19)
    at org.eclipse.gef.editparts.AbstractEditPart.createChild(AbstractEditPart.java:273)
    at com.workday.sh.sa.editor.assembly.impl.editparts.SwimlaneEditPart.refreshChildren(SwimlaneEditPart.scala:155)
```

### Why It Happens
Workday Studio’s graphical editor is built on the Eclipse Graphical Editing Framework (GEF) and Eclipse Modeling Framework (EMF), with custom Scala edit-part factories.
1. When `assembly-diagram.xml` is opened, GEF reads all `<elements href="..."/>` and asks EMF to resolve the target object in `assembly.xml`.
2. If EMF cannot resolve the target (either because the element ID does not exist in `assembly.xml`, or because the target element type does not register an XML `ID` attribute in EMF), EMF returns a generic `org.eclipse.emf.ecore.impl.EObjectImpl` proxy.
3. `Factory.scala` executes pattern matching:
   ```scala
   def createEditPart(context: EditPart, model: Any): EditPart = model match {
     case m: WorkdayIn => new WorkdayInEditPart(m)
     case m: AsyncMediation => new AsyncMediationEditPart(m)
     case m: SendError => new SendErrorEditPart(m)
     case m: Swimlane => new SwimlaneEditPart(m)
     // Notice: No case for EObjectImpl!
   }
   ```
4. Because there is no wildcard or `case _: EObjectImpl` handler, Scala throws `scala.MatchError` and the graphical canvas crashes.

### The Two Golden Rules to Eliminate `scala.MatchError`
* **Rule A**: Never add an element to `assembly-diagram.xml` before it is declared in `assembly.xml`.
* **Rule B**: For top-level `<cc:send-error>`, use the exact feature map index fragment:
  ```xml
  <elements href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
  ```
  *(or verify the exact position if other top-level elements precede it).*

---

## 3. Structural Rules for `assembly.xml`

### 1. Root & Namespace Declarations
Every `assembly.xml` must declare standard schemas and namespaces:
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
 
    <cc:assembly id="WorkdayAssembly" version="2025.21">
        ...
    </cc:assembly>
</beans>
```

### 2. Entry Points (`<cc:workday-in>`)
* **Scheduled Batch**: Requires datetime prompts (e.g., `Updated From` and `Updated To`).
* **Ad-Hoc Manual**: Stacked as a secondary entry point (`StartHere-Adhoc`) sharing services.
* **Real-Time BP-Triggered**: Eliminates date ranges; accepts a single event identifier (e.g., `Action_Event_WID`).

### 3. Dynamic API Versioning
Never hard-code Web Service API versions. Declare an Integration Attribute and resolve it dynamically in the first mediation step:
```xml
<cc:eval id="EvalInit">
    <cc:expression>props['apiversion'] = intsys.getAttribute('API Version')</cc:expression>
    <cc:expression>if (props['apiversion'] == empty || props['apiversion'] == '') { props['apiversion'] = 'v45.0'; }</cc:expression>
</cc:eval>
```

### 4. Robust Currency & Parsing Fallbacks
Never rely on single character substring extractions (e.g., parsing amounts using `substring-after(..., '₹')`). Always implement a 3-tier fallback in MVEL/Java expressions:
```java
props['Amount'] = parts[0].xpath("substring-after(..., '₹')");
if (props['Amount'] == empty || props['Amount'] == '') {
    props['Amount'] = parts[0].xpath("wd:Report_Data/wd:Report_Entry/wd:Applied_Amount");
}
if (props['Amount'] == empty || props['Amount'] == '') {
    props['Amount'] = parts[0].xpath("wd:Report_Data/wd:Report_Entry/wd:Total_Amount");
}
```

### 5. Standard Error Handling Pattern
Every asynchronous mediation that performs calls, calculations, or splits must include:
* `handle-downstream-errors="true"`
* `<cc:send-error id="..." routes-to="..."/>` pointing to a dedicated error handler.
* An error increment counter: `props['Error.Count'] = props['Error.Count'] + 1`.

---

## 4. Visual Architecture for `assembly-diagram.xml`

### Canvas Grid Model
```
Y-Coord   Row Description                    Visual Pattern
───────────────────────────────────────────────────────────────────────────────────────────────────
y = 21    Top-Right Global Error Block       [SendError] ──► [Note-Error]
y = 50    Row 1: Main Pipeline               [StartHere] ──► [Init] ──► [Local-Outs] ──► [Store-Log] ──► [PIM]
y = 260   Row 2: Pre-fetch / Lookup Tables   [Local-In] ──► [Sub-Lane: Med + Error] ──► [Out-Rest] ──► [Copy]
y = 500   Row 3: Primary Driver & Split      [Local-In] ──► [Sub-Lane] ──► [Out-Rest] ──► [Splitters]
y = 760   Row 4: Verification & Mutation     [Local-In] ──► [Sub-Lane] ──► [Out-Soap] ──► [Log]
y = 1040  Row 5: Detail & Routing            [Local-In] ──► [Sub-Lane] ──► [Calculation] ──► [Route]
y = 1320  Row 6: Processing / Submission     [Local-In] ──► [Sub-Lane: Xslt + Error] ──► [Out-Soap] ──► [Log]
```

### Standard Horizontal Spacing (X-Axis Steps)
* Node width standard: ~120px to 140px.
* Step offset $\Delta x$: 180px to 210px between sequential nodes.
* X progression: `50` $\rightarrow$ `230` $\rightarrow$ `440` $\rightarrow$ `650` $\rightarrow$ `860` $\rightarrow$ `1070` $\rightarrow$ `1280` $\rightarrow$ `1490` $\rightarrow$ `1700`.

### Standard Vertical Sub-Lane Spacing (Y-Axis Steps)
* Parent mediation step: $y = N$ (e.g., $y = 520$).
* Child error handling step directly below: $y = N + 100$ (e.g., $y = 620$).
* Error sub-swimlane container: encapsulates both with `orientation="VERTICAL"`.

---

## 5. Swimlane Indexing & Nesting Rules

In Workday Studio, swimlanes are referenced by zero-based index in the order they are declared in `assembly-diagram.xml`.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SWIMLANE INDEXING TAXONOMY                            │
├──────────────┬────────────────────────────────────────┬─────────────────────────┤
│ Index        │ Type & Role                            │ Referenced By           │
├──────────────┼────────────────────────────────────────┼─────────────────────────┤
│ Swimlane 0   │ Horizontal: Global Error Handler       │ Standalone (Top-Right)  │
│ Swimlane 1   │ Horizontal: Main Pipeline              │ Parent Container        │
│ Swimlane 2   │ Vertical: Init Sub-Lane                │ Child of Swimlane 1     │
│ Swimlane 3   │ Horizontal: Extract Report A           │ Parent Container        │
│ Swimlane 4   │ Vertical: Report A Error Sub-Lane      │ Child of Swimlane 3     │
│ Swimlane 5   │ Horizontal: Extract Report B           │ Parent Container        │
│ Swimlane 6   │ Vertical: Report B Error Sub-Lane      │ Child of Swimlane 5     │
│ Swimlane 7   │ Horizontal: Driver Extraction          │ Parent Container        │
│ Swimlane 8   │ Vertical: Call Error Sub-Lane          │ Child of Swimlane 7     │
│ Swimlane 9   │ Vertical: Result Error Sub-Lane        │ Child of Swimlane 7     │
│ Swimlane 10  │ Vertical: Line Error Sub-Lane          │ Child of Swimlane 7     │
│ Swimlane 11  │ Horizontal: Verification & SOAP Cancel │ Parent Container        │
│ Swimlane 12  │ Vertical: Query Error Sub-Lane         │ Child of Swimlane 11    │
│ Swimlane 13  │ Vertical: Cancel Error Sub-Lane        │ Child of Swimlane 11    │
│ Swimlane 14  │ Horizontal: Detail Extraction & Route  │ Parent Container        │
│ Swimlane 15  │ Vertical: Detail Call Error Sub-Lane   │ Child of Swimlane 14    │
│ Swimlane 16  │ Vertical: Calculation Error Sub-Lane   │ Child of Swimlane 14    │
│ Swimlane 17  │ Vertical: Routing Outcomes Sub-Lane    │ Child of Swimlane 14    │
│ Swimlane 18  │ Horizontal: Record Submission          │ Parent Container        │
│ Swimlane 19  │ Vertical: SOAP Submit Error Sub-Lane   │ Child of Swimlane 18    │
└──────────────┴────────────────────────────────────────┴─────────────────────────┘
```

### Exact Parent-Child Syntax
In parent swimlane `N`, reference the nested vertical child `M` using:
```xml
<elements href="#//@swimlanes.M"/>
```

---

## 6. Complete Starter Templates

### Template A: Standard `assembly.xml`
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
 
    <cc:assembly id="WorkdayAssembly" version="2025.21">
        <!-- Entry Point -->
        <cc:workday-in id="StartHere" routes-to="Initialization">
            <cc:integration-system name="INT_Template_System">
                <cloud:attribute-map-service name="INT_Attributes">
                    <cloud:attribute name="API Version">
                        <cloud:type>
                            <cloud:class-report-field description="Web Service API Version" type="WID" singular="true">643097491d2410000a9da38c99860026</cloud:class-report-field>
                        </cloud:type>
                    </cloud:attribute>
                </cloud:attribute-map-service>
            </cc:integration-system>
        </cc:workday-in>

        <!-- Initialization -->
        <cc:async-mediation id="Initialization" routes-to="Store-Log">
            <cc:steps>
                <cc:eval id="Eval">
                    <cc:expression>props['Error.Count'] = 0</cc:expression>
                    <cc:expression>props['apiversion'] = intsys.getAttribute('API Version')</cc:expression>
                    <cc:expression>if (props['apiversion'] == empty || props['apiversion'] == '') { props['apiversion'] = 'v45.0'; }</cc:expression>
                </cc:eval>
                <cc:cloud-log id="CloudLog" level="info" message="Process Initialized" output-file-type="HTML"/>
            </cc:steps>
        </cc:async-mediation>

        <!-- Audit Store & PIM -->
        <cc:async-mediation id="Store-Log" routes-to="Success-PIM">
            <cc:steps>
                <cc:cloud-log id="CloudLog" level="info" message="Execution Completed" output-file-type="HTML"/>
                <cc:store id="Store" input="variable" input-variable="cloud-log-content" createDocumentReference="true" expiresIn="P180D" title="Audit_Log.html"/>
            </cc:steps>
        </cc:async-mediation>
        <cc:local-out id="Success-PIM" execute-when="props['Error.Count'] == 0" routes-response-to="Error-PIM" endpoint="vm://wcc/PutIntegrationMessage">
            <cc:set name="is.message.severity" value="'INFO'"/>
            <cc:set name="is.message.summary" value="'Integration executed successfully.'"/>
        </cc:local-out>
        <cc:local-out id="Error-PIM" execute-when="props['Error.Count'] > 0" endpoint="vm://wcc/PutIntegrationMessage">
            <cc:set name="is.message.severity" value="'ERROR'"/>
            <cc:set name="is.message.summary" value="'Integration completed with errors.'"/>
        </cc:local-out>

        <!-- Top-Level Global Error Handler -->
        <cc:send-error id="global-error-handler" rethrow-error="false" routes-to="Note-Error"/>
        <cc:local-out id="Note-Error" endpoint="vm://wcc/PutIntegrationMessage">
            <cc:set name="is.message.severity" value="'CRITICAL'"/>
            <cc:set name="is.message.summary" value="context.errorMessage"/>
        </cc:local-out>
    </cc:assembly>
</beans>
```

### Template B: Matching `assembly-diagram.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wdnm:Diagram xmlns:wdnm="<redacted URL>">
  <element href="assembly.xml#WorkdayAssembly"/>

  <!-- Global Error Handler (Top-Right) -->
  <visualProperties x="1610" y="21">
    <element href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
  </visualProperties>
  <visualProperties x="1780" y="21">
    <element href="assembly.xml#Note-Error"/>
  </visualProperties>

  <!-- Row 1 Main -->
  <visualProperties x="50" y="60">
    <element href="assembly.xml#StartHere"/>
  </visualProperties>
  <visualProperties x="240" y="60">
    <element href="assembly.xml#Initialization"/>
  </visualProperties>
  <visualProperties x="450" y="60">
    <element href="assembly.xml#Store-Log"/>
  </visualProperties>
  <visualProperties x="660" y="35">
    <element href="assembly.xml#Success-PIM"/>
  </visualProperties>
  <visualProperties x="660" y="95">
    <element href="assembly.xml#Error-PIM"/>
  </visualProperties>

  <!-- Connections -->
  <connections type="routesTo" sourceAnchorX="0.9669421487603306" sourceAnchorY="0.43243243243243246" targetAnchorX="0.06060606060606061" targetAnchorY="0.3783783783783784">
    <source href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
    <target href="assembly.xml#Note-Error"/>
  </connections>
  <connections type="routesTo" sourceAnchorX="1.0" sourceAnchorY="0.5" targetAnchorX="0.0" targetAnchorY="0.5">
    <source href="assembly.xml#StartHere"/>
    <target href="assembly.xml#Initialization"/>
  </connections>
  <connections type="routesTo" sourceAnchorX="1.0" sourceAnchorY="0.5" targetAnchorX="0.0" targetAnchorY="0.5">
    <source href="assembly.xml#Initialization"/>
    <target href="assembly.xml#Store-Log"/>
  </connections>
  <connections type="routesTo" sourceAnchorX="1.0" sourceAnchorY="0.3" targetAnchorX="0.0" targetAnchorY="0.5">
    <source href="assembly.xml#Store-Log"/>
    <target href="assembly.xml#Success-PIM"/>
  </connections>
  <connections type="routesResponseTo" sourceAnchorX="0.5" sourceAnchorY="1.0" targetAnchorX="0.5" targetAnchorY="0.0">
    <source href="assembly.xml#Success-PIM"/>
    <target href="assembly.xml#Error-PIM"/>
  </connections>

  <!-- Swimlanes -->
  <swimlanes x="1610" y="21" name="Global Error Handler" topBorderColor="255">
    <elements href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
    <elements href="assembly.xml#Note-Error"/>
  </swimlanes>

  <swimlanes x="31" y="9" font="Arial|14|1" fontColor="1" name="INT-Main" labelAlignment="LEFT">
    <elements href="assembly.xml#StartHere"/>
    <elements href="#//@swimlanes.2"/>
    <elements href="assembly.xml#Store-Log"/>
    <elements href="assembly.xml#Success-PIM"/>
    <elements href="assembly.xml#Error-PIM"/>
  </swimlanes>
  <swimlanes x="220" y="50" name="Swimlane" orientation="VERTICAL">
    <elements href="assembly.xml#Initialization"/>
  </swimlanes>

</wdnm:Diagram>
```

---

## 7. Developer Pre-Deployment Checklist

Before committing or testing any Workday Studio build:

- [ ] **Cross-Reference Check**: For every `<element href="assembly.xml#ID"/>` in `assembly-diagram.xml`, verify that `<cc:... id="ID">` exists in `assembly.xml`.
- [ ] **Global Error Handler Check**: Verify that `global-error-handler` is referenced via `assembly.xml#//@beans/@mixed.1/@mixed.39` and NOT `#global-error-handler`.
- [ ] **Outer Swimlane Audit**: Confirm there is no master swimlane wrapping other horizontal swimlanes.
- [ ] **Swimlane Indexing Audit**: Verify that each `#//@swimlanes.N` references the exact zero-based position of the intended child vertical swimlane.
- [ ] **Anchor Standard**: Ensure forward links use `(1.0, 0.5) -> (0.0, 0.5)` and error drops use `(0.5, 1.0) -> (0.5, 0.0)`.
- [ ] **Dynamic API Versioning**: Confirm Web Service version is read from Integration Attributes with a sensible default (`v45.0`+).
- [ ] **Rupee / Currency Hardening**: Ensure all XPath substring extractions have fallbacks to raw numerical/amount elements.
