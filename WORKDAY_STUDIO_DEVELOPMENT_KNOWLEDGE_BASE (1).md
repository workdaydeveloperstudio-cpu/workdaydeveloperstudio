# Workday Studio Development Knowledge Base

**For:** Yogarajan Venkataraman  
**Files:** `assembly.xml`, `assembly-diagram.xml`, `*.xsl` / `*.xslt`  
**Version:** 1.1  
**Updated:** 2026-11-13, Asia/Calcutta

> This consolidated guide replaces conflicting instructions in the old `WORKDAY_STUDIO_GENERATION_SPECIFICATION.md` and earlier replies. It is not an official Workday schema or a zero-error guarantee. Current requirements, installed schemas/metamodel and actual project files take precedence.
>
> The user confirmed local application of namespace, swimlane-attribute and handler-visual patches. A final successful diagram open/save/reopen and end-to-end integration run have **not** been confirmed.
>
> **Delivery protection:** a saved draft was inspected and found to contain automatically redacted namespace URLs. This version records namespace host/path components separately and constructs values locally without embedding raw HTTP namespace URLs. Do not copy redacted values or unresolved template tokens into XML.
>
> **Version 1.1 addition:** the two latest INT0016 uploads are registered as a matched reference pair for future projects in section 14. The guide contains their source links and reuse instructions, not their complete XML contents. Retain the original sample files alongside this MD.

## Contents

1. Evidence and sources
2. Current incident and applied fixes
3. Rules for future development
4. Error-to-fix register
5. Namespace construction and XML envelopes
6. Assembly patterns
7. Diagram patterns and reference rules
8. XSLT patterns and error prevention
9. Project-specific historical lessons
10. Safe local diagnostics and repair procedures
11. Validation and release checklist
12. Web / Workday Community research
13. Future prompt, change log and open work
14. INT0016 matched reference pair for future Studio projects

---

## 1. Evidence and sources

### Evidence labels

- **OBSERVED:** visible in a source read for this guide or user-provided error/output.
- **SAMPLE:** demonstrated by the reference pair; not an exhaustive version-independent specification.
- **APPLIED:** local edit confirmed by user output, not automatically Studio/runtime success.
- **HISTORICAL:** recorded in the original MD, prior conversation or user-provided historical notes; underlying tenant/log evidence may not be independently available.
- **RECOMMENDED:** preventive guidance/proposed test, not work already executed.
- **PENDING:** further schema/model/editor/build/runtime evidence needed.

### Source register

| ID | Source | Authority / limits |
|---|---|---|
| S1 | [INT0016_sample_assembly.txt — latest reference upload](file://user/fileupload/461cd97c-e42c-4b4f-a8b7-f2639e92b348) | XML content despite .txt extension. Reference namespaces, services, expressions, routing, retrieval and XSLT invocation; not a complete XSD. Referenced XSL files not attached. Paired with S2. |
| S2 | [INT0016_sample_assembly-diagram.txt — latest reference upload](file://user/fileupload/d770617c-4244-47c2-bdde-2f0f7fe3a802) | XML content despite .txt extension. Reference notation, structural fragments and swimlanes. Indices belong to that exact model. Paired with S1. |
| S3 | [Uploaded INT0099 assembly](file://user/fileupload/6476484f-69f3-46f7-b4c6-2e22c192c956) | Actual generated source and implementation risks; no runtime pass established. |
| S4 | [Uploaded INT0099 diagram](file://user/fileupload/dea17e52-e277-486b-b3de-478b7effc610) | Pre-patch snapshot with wrong namespace/attributes/handler reference. Do not overwrite repaired local files with it. |
| S5 | [Original specification](file://user/fileupload/5f2c7dbc-7e86-4de3-93d2-e8324986f1b2) | Historical register, but contains redacted templates and overbroad claims; not an executable template. |
| S6 | User-pasted errors and PowerShell output in this conversation | Actual failure stages, applied changes, backup paths and handler route; not executed independently by the assistant. |
| S7 | User-provided historical notes in this conversation | INT0076 authentication, INT0047 PO mapping, Ad Hoc Bank XSLT and layout preferences; project history, not current tenant verification. |
| S8 | [Uploaded knowledge base, version 1.0](file://user/fileupload/97ae4219-9e46-4d0d-b5f7-0e9909190fc4) | Baseline document for this version 1.1 update. Existing troubleshooting content retained; source register, reuse prompt and reference guidance updated. |

For traceability, version 1.0 used earlier sample-upload links: [earlier sample assembly](file://user/fileupload/d8f67f2c-f47d-4138-94e6-fbb0f48e9a6b) and [earlier sample diagram](file://user/fileupload/1a7cfe76-eb5f-4b52-b39f-46a4e6422c6f). The latest S1/S2 uploads above were read through their closing tags for the version 1.1 reference update. No byte-for-byte equivalence between upload revisions is claimed.

The version 1.0 guide recorded that its S1–S4 were read through their closing tags. Earlier claims of an unavoidable 8,000-character reading limit should not guide future work.

No actual applicant request stylesheet was uploaded. This guide does not claim to have reviewed/corrected `PutApplicantRequest.xsl` itself.

Authority order: current requirements and installed model → working same-build sample → current source/diffs/reproducible outcomes → verified matching official/Community evidence → this guide and memories. Memory cannot override actual file/schema evidence.

---

## 2. Current incident and applied fixes

Project: **test_Rovo**. Integration: **INT0099-Load Applicant from Alight**. Files: `ws/WSAR-INF/assembly.xml`, `ws/WSAR-INF/assembly-diagram.xml`. Reference: INT0016. Exact installed Studio version/build: **not supplied**.

| Failure | Evidence | Action / status |
|---|---|---|
| `None.get` at GmfElement / Diagram / GMFModel.startElement / GMFReader | Namespace in S4 differs from S2 | Namespace corrected locally; APPLIED. A later EMF error followed. Not a universal cause of every None.get. |
| `Feature 'width' not found`, line 269/column 192 | Unsupported width on generated swimlane | Width/height/fgColor removed from swimlanes; APPLIED. Only width was explicitly rejected; other removals aligned with sample. |
| `scala.MatchError` with proxy `assembly.xml#GlobalErrorHandler` | ID-style handler reference unresolved during edit-part creation | Three broken diagram uses removed; APPLIED. Temporary visual workaround, not permanent fragment repair. |
| After workaround | Actual handler routes-to Note-Error; three visual references removed | Actual handler retained according to output; final editor/runtime tests PENDING. |

A changed error stage indicates progress, not complete correctness.

### Exact namespace difference, without raw URL embedding

Both use HTTP. Correct host labels: `workday`, `com`; correct path segments: `studio`, `editors`, `notation`. Incorrect generated host adds `www`; incorrect path adds `1.0`. Build exact values using section 5.

Actual handler retained:

```xml
<cc:send-error id="GlobalErrorHandler" rethrow-error="false" routes-to="Note-Error"/>
```

The workaround intentionally removes its icon and outgoing arrow only. Missing artwork is not removed runtime logic, but runtime execution still requires testing.

### User-confirmed backups

```text
%LOCALAPPDATA%\WorkdayStudio-Backups\20261113-071837-599
%LOCALAPPDATA%\WorkdayStudio-Backups\20261113-072308-878
%LOCALAPPDATA%\WorkdayStudio-Backups\20261113-074958-621
```

These correspond to namespace, swimlane and proxy-visual changes. Inspect backup contents before restoration; later backups may include earlier patches. Preserve current repairs; do not run old broad scripts that also change services, expressions, XSL paths and business logic while testing notation.

---

## 3. Rules for future development

1. Read actual current files completely and use a matching working sample/build. Use the INT0016 pair registered in section 14 as a reference, not as an automatically validated new project.
2. Never invent namespace URIs, schema attributes, service APIs, endpoint names, request schemas or EMF indices.
3. Fix one layer per test: XML → schema/notation → model references → layout → expressions/resources → runtime.
4. Close Studio before external writes. Back up outside the project and inspect exact diffs.
5. Diagram-only repair must not change assembly logic, XSLT, services, endpoints or security.
6. Existing XML id does not prove `#thatId` resolves for every EMF type.
7. Never transfer `@mixed.N` between projects or compute it from XML line/tag counts.
8. Use semantic consistency, not a strict every-ID-in-three-sections rule.
9. No fake production input, fixed duplicate-email lists or sequence 1 fallbacks.
10. Do not assume Groovy, Java and MVEL syntax are interchangeable.
11. Validate intended XSLT processor/version/extensions and actual resource paths.
12. Version 2022.10 is observed in this sample, not a universal mandatory version.
13. Preserve group headers, hierarchy, spacing and requested column layouts.
14. No zero-error claims; report actual validation levels and untested items.
15. Update this MD whenever a source-supported error/fix/test result is established.
16. Keep credentials, tokens, personal data and copied tenant WIDs out of reusable examples.
17. Keep both original INT0016 sample files together with this MD. Adapt project-specific identifiers/resources and validate the new model before reuse; upload links alone are not a durable local archive.

### Earlier advice explicitly retracted

| Earlier claim | Correct rule |
|---|---|
| Similar-looking namespaces are equivalent | Identity must match exactly. |
| Initial None.get proves global handler reference is the cause | Early trace did not identify the absent value; later proxy error did identify the handler reference. |
| Every XML id is an EMF ID | Not established; handler-ID reference failed here. |
| Always use mixed.39 or mixed.71 | Current-model generated/verified fragment only. |
| Every ID must appear in positions, connections and swimlanes | Internal steps, nested ports and terminal components differ. |
| Remove every XML comment | No supplied error proves a blanket comment prohibition. |
| Sequence/retrieval services are invalid | Both are in S1; inspect placement/configuration. |
| execute-when works on every component | Attributes differ; async sample uses execute-steps-when. |
| Ban all conditionals/typed constructs in every case | Diagnose exact language/version; sample includes conditional expressions. |
| Ban every XSLT subfolder or cc:xslt | S1 demonstrates xslt-plus; other paths/processors need target verification. |
| Force every assembly to version 2022.10 | Use supported target version and actual dependency evidence. |
| Stylesheet indentation alone proves bad output whitespace | Inspect payload and normalize intentionally. |
| Work Order mapping fixes closed-history PO rejection | Mapping requirement and target lifecycle are separate. |
| Proxy/update warnings prove cache corruption | Correlate current action/time/exception first. |
| Saved artifact is already validated | Creation is not schema, EMF, editor or runtime validation. |

---

## 4. Error-to-fix register

Coverage: errors recorded in this conversation, original MD and supplied history, not all possible Studio errors.

### XML / diagram / EMF

| Error / risk | Correction | Evidence / limits |
|---|---|---|
| None.get at GMFReader / Diagram constructor | Match exact sample namespace, verify actual saved local file, reopen | OBSERVED; APPLIED, later failure moved to different stage. Not universal diagnosis. |
| Feature width not found | Remove unsupported width on that swimlane | OBSERVED line 269/192; APPLIED. |
| Swimlane height/fgColor | Omit in this sample-based template unless model validates them | Removed for alignment, not separately proven unsupported in every version. |
| MatchError + GlobalErrorHandler proxy | Permanent current-model fragment; temporary exact visual/edge/membership removal | OBSERVED; workaround applied, final editor pass pending. |
| Other unresolved proxy/EObject | Inspect exact URI, assembly parse, target existence and semantic type | Don't delete all handlers/proxies. |
| Invalid nested lane index | Recompute after reorder; verify targets/cycles | HISTORICAL and sample pattern. |
| SAX/XML error around URL/redacted namespace | Rebuild exact values locally, reject sanitized XML, parse actual bytes | HISTORICAL; redacted S5 visible. |
| Raw ampersand/less-than in XML | Proper entity escaping/XML serializer | General XML syntax. |
| Same error after patch | Check Resource → Location, namespace/content/hash/time and newest error | Could be wrong copy, unapplied edit or remaining defect. |
| Overlap/missing artwork | Inspect hierarchy/defaults/coordinates separately from runtime logic | Layout issue, not necessarily runtime failure. |

### Assembly schema / expressions / resources

| Error / risk | Correction / prevention | Evidence / limits |
|---|---|---|
| execute-when rejected on async-mediation | Use supported execute-steps-when if correct for requirement | S1; skipping steps need not skip downstream route. |
| execute-when rejected on sub-route | Use supported strategy/condition | Historical; exact XSD required. |
| Route missing strategy | Sample has all-strategy or doc-iterator in appropriate routes | Don't use all-strategy for exclusive choice. |
| Splitter rejects routes-to/xpath or requires route name | Name and routes-to on sub-route, XPath on supported splitter strategy | S1 shows xml-stream-splitter; not every alternate type verified. |
| XSLT processor under assembly rejected | Supported processor inside mediation steps | Sample; target XSD authority. |
| cloud:default under attribute rejected | Default shown under param; validate attribute value structure | S1/S3 comparison, target XSD needed. |
| cloud:number rejected | Distinguish simple-type number from value element cloud:numeric in S3 | Verify exact schema. |
| simple-type string rejected | Recorded string-like alternative: text | Historical list boolean/date/datetime/number/text; exhaustive enum needs XSD. |
| Sequence/retrieval rejected at a location | Keep valid type, correct order/configuration | Both in S1; full order including report-service not established by S1. |
| Invented sequence API/fallback 1 | Sample lp.getSequencedValue with configured sequencer | Prevent hidden config fault/collision. |
| Invented attribute API | Sample intsys.getAttribute | Alternate APIs need verified documentation. |
| MVEL no viable alternative at input String | Simplify exact failing expression, test intended language/compiler | Historical; original source/version not independently present. |
| Class/import problem | Verify actual class availability/imports; qualify where supported | Preventive, not reproduced compiler result here. |
| XSL file cannot be found | URL/case/suffix/packaging/refresh/context | Subfolder alone isn't proven invalid. |
| cc:log cannot have character children | Component-supported structured message content | Historical; don't invent replacement logging processor. |
| Duplicate collection service name | Identify true conflict scope, use specific names where needed | Not universal uniqueness across all projects. |
| Classloader/version mismatch | Actual Studio/Java/dependencies/collection evidence | Don't force sample version blindly. |
| Missing inputs become success | No fake fallback; update errors and continuation state | Risks visible S3; runtime tests needed. |

### Transport / reports / target systems

| Error | Correction / investigation | Scope |
|---|---|---|
| RaaS 404 with service/null | Exact alias, reject null, actual endpoint/report | Historical S5; not every 404. |
| HTTP 400 Invalid Parm | Actual Prompt XML Alias / View URLs, URL-encoded values | Historical S5. |
| HTTP 500 Task not authorized | Effective ISU report/data-source/field/domain access; authorized pending changes | Historical S5, no security writes here. |
| ISU not in ownership picker | Verify eligibility/security; sharing may be alternative | Historical possibilities, not one proven missing domain. |
| Real-time/delta mismatch | Match trigger/report/prompt contract, not merely names | Historical S5. |
| Success despite extract failure | Count caught errors and gate final messages | Historical and S3 review. |
| INT0076 401 after refresh | Project-specific authentication activation/ISSG/tenant checks | Historical S7. |
| Maximo BMXAA4137E | Actual status/history/lifecycle, eligible test PO | Historical S7; not inherently XSLT fault. |
| Duplicate icon warnings | Previously nonblocking in one INT0076 build | Don't generalize to every plugin/icon failure. |

### PowerShell / delivery

| Error | Remedy |
|---|---|
| else treated as command | Else arrived separately; paste whole reviewed block, use wrapper/no-else style. |
| Continuation prompt >> | Incomplete quotes/brackets; Ctrl+C and repaste complete block. |
| Missing path | Use actual Eclipse Resource → Location and literal path handling. |
| Redacted URL inside saved XML/MD namespace | Invalid example; use component dictionary/local generation below. |
| Old download inaccessible/401 | Access limitation, not content validation; don't claim inspected. |

---

## 5. Namespace construction and XML envelopes

### 5.1 Canonical components

Use HTTP scheme; join host labels with period, path segments with slash. Host/path components are deliberately separated to survive the observed delivery sanitizer. These values are observed in S1/S2 except standard XSLT/XS declarations, which are general XML/XSLT syntax and not retrieved from public documentation in this session.

| Name | Host labels | Path segments / fixed URN |
|---|---|---|
| Spring beans | www, springframework, org | schema, beans |
| Atom | www, w3, org | 2005, Atom |
| Cape Clear assembly | www, capeclear, com | assembly, 10 |
| Workday cloud | — | urn:com.workday/esb/cloud/10.0 |
| SOAP 1.1 envelope | schemas, xmlsoap, org | soap, envelope, empty-final-segment |
| PICOF | — | urn:com.workday/picof |
| Workday service | — | urn:com.workday/bsvc |
| XML Schema instance | www, w3, org | 2001, XMLSchema-instance |
| Studio notation | workday, com | studio, editors, notation |
| XSLT | www, w3, org | 1999, XSL, Transform |
| XML Schema types | www, w3, org | 2001, XMLSchema |

Spring legitimately includes www. This notation sample does not. Do not globally remove www.

### 5.2 Read-only generator for exact namespace values and envelopes

This PRINTS values and envelope illustrations; writes no project files. It is not a complete runnable integration. Keep the closing PowerShell here-string markers at the start of their lines.

```powershell
& {
    function New-XmlNamespace {
        param([string[]]$Labels, [string[]]$Segments)
        $slash = [string][char]47
        return ('http' + ':' + $slash + $slash +
                ($Labels -join '.') + $slash + ($Segments -join $slash))
    }
    $ns = @{
        beans = (New-XmlNamespace -Labels @('www','springframework','org') -Segments @('schema','beans'))
        atom = (New-XmlNamespace -Labels @('www','w3','org') -Segments @('2005','Atom'))
        cc = (New-XmlNamespace -Labels @('www','capeclear','com') -Segments @('assembly','10'))
        cloud = 'urn:com.workday/esb/cloud/10.0'
        env = (New-XmlNamespace -Labels @('schemas','xmlsoap','org') -Segments @('soap','envelope',''))
        pi = 'urn:com.workday/picof'
        wd = 'urn:com.workday/bsvc'
        xsi = (New-XmlNamespace -Labels @('www','w3','org') -Segments @('2001','XMLSchema-instance'))
        notation = (New-XmlNamespace -Labels @('workday','com') -Segments @('studio','editors','notation'))
        xsl = (New-XmlNamespace -Labels @('www','w3','org') -Segments @('1999','XSL','Transform'))
        xs = (New-XmlNamespace -Labels @('www','w3','org') -Segments @('2001','XMLSchema'))
    }
    $ns.GetEnumerator() | Sort-Object Name | Format-Table -AutoSize
    Write-Output '--- assembly envelope: sample version, not universal ---'
    @"
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="$($ns.beans)" xmlns:beans="$($ns.beans)"
  xmlns:atom="$($ns.atom)" xmlns:cc="$($ns.cc)"
  xmlns:cloud="$($ns.cloud)" xmlns:env="$($ns.env)"
  xmlns:pi="$($ns.pi)" xmlns:wd="$($ns.wd)" xmlns:xsi="$($ns.xsi)">
  <cc:assembly id="WorkdayAssembly" version="2022.10">
    <!-- Add supported components for your actual build. -->
  </cc:assembly>
</beans>
"@
    Write-Output '--- minimal diagram envelope ---'
    @"
<?xml version="1.0" encoding="UTF-8"?>
<wdnm:Diagram xmlns:wdnm="$($ns.notation)">
  <element href="assembly.xml#WorkdayAssembly"/>
</wdnm:Diagram>
"@
    Write-Output '--- XSLT envelope: compatible processor required ---'
    @"
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="2.0" xmlns:xsl="$($ns.xsl)"
  xmlns:xs="$($ns.xs)" xmlns:wd="$($ns.wd)" exclude-result-prefixes="xs">
  <xsl:output method="xml" encoding="UTF-8" indent="yes"/>
  <!-- Templates must match actual input and target schema. -->
</xsl:stylesheet>
"@
}
```

The fragments below assume enclosing document declarations generated above. They are not standalone XML documents. Scripts in this guide have not been executed by the assistant.

---

## 6. Assembly patterns

### Services and schema order

S1 main-block order:

```text
param → attribute-map-service → sequence-generator-service
      → retrieval-service → service-reference
```

S1 has no report-service in that block. Earlier claims of a universal order including report-service require installed XSD verification.

```xml
<cloud:sequence-generator-service name="Example-Batch">
  <cloud:sequencer>Example-BatchID</cloud:sequencer>
</cloud:sequence-generator-service>
<cloud:retrieval-service name="Example-Retrieval"/>
<cloud:service-reference name="Example-FieldOverride"/>
```

These are adapted illustrative names. Generic service-reference is not automatically a sequence-generator replacement. Verify report placement/alias/WID/tenant/prompts; don't copy sample identifiers.

### Parameters and attributes

```xml
<cloud:param name="Validation Mode">
  <cloud:type><cloud:simple-type>boolean</cloud:simple-type></cloud:type>
  <cloud:default><cloud:boolean>true</cloud:boolean></cloud:default>
</cloud:param>
<cloud:attribute-map-service name="Example-Attributes">
  <cloud:attribute name="Document Retention Days">
    <cloud:type><cloud:simple-type>number</cloud:simple-type></cloud:type>
  </cloud:attribute>
</cloud:attribute-map-service>
```

Configure real target values. For embedded cloud:value used in S3, validate target XSD; do not place default arbitrarily.

### Expressions and APIs demonstrated by sample

Inside mediation steps:

```xml
<cc:eval id="LP">
  <cc:expression>props['lp.validation.mode'] = lp.getSimpleData('Validation Mode')</cc:expression>
</cc:eval>
<cc:eval id="Attr">
  <cc:expression>props['attr.doc.retention.days'] = intsys.getAttribute('Document Retention Days')</cc:expression>
</cc:eval>
<cc:eval id="Prop">
  <cc:expression>props['continue.processing'] = true</cc:expression>
  <cc:expression>props['batch.id'] = lp.getSequencedValue('Example-BatchID')</cc:expression>
</cc:eval>
```

Names must match configuration. Prefer small testable statements. Do not assume def/interpolation/Elvis/Java loops/unqualified classes/catch syntax works across processors. Don't ban all conditionals: S1 uses if/else in set expressions. Interpolation @{...} is demonstrated in specific attribute contexts, not universally. Check actual null/boolean types. Escape XML metacharacters (e.g. `&amp;&amp;`, `&lt;`). Don't swallow sequence errors and substitute 1.

### Routing and splitter

```xml
<cc:route id="ExampleRoute">
  <cc:all-strategy/>
  <cc:sub-route name="Import" routes-to="PrepareRequest"/>
  <cc:sub-route name="Audit" routes-to="PrepareAudit"/>
</cc:route>
<cc:splitter id="ExampleSplitter">
  <cc:sub-route name="SubRoute" routes-to="ProcessRecord"/>
  <cc:xml-stream-splitter xpath="Root/Row"/>
</cc:splitter>
```

Targets must exist. All-strategy is not exclusive choice. Use real message paths/namespaces. Do not put routes-to/xpath on unsupported parent locations. S1 demonstrates execute-when on local-out and execute-steps-when on async-mediation; skipping steps need not skip downstream routing. Distinguish routes-to from routes-response-to.

### Error handlers / completion

```xml
<cc:async-mediation id="PrepareRequest" routes-to="CallService"
                    handle-downstream-errors="true">
  <cc:steps><!-- Supported steps. --></cc:steps>
  <cc:send-error id="SendError" routes-to="CallCriticalError"/>
</cc:async-mediation>
```

S1 repeats nested SendError IDs. Don't impose global uniqueness on all id attributes without scope/model knowledge; top-level routing must be unambiguous.

Initialize error state early, count caught failures, avoid null-counter masking, define warning semantics and prevent contradictory final messages. Severity alone doesn't prove counters/runtime status updated. Distinguish validated/attempted/loaded/failed/skipped. Test runtime handler separately from icon visibility.

### Retrieval / CSV

```xml
<cc:eval id="ReadRetrievedDocument">
  <cc:expression>da.toVar(0,&quot;wd.retrieve.variable&quot;)</cc:expression>
</cc:eval>
<cc:csv-to-xml id="CsvToXml" output="variable" output-variable="input.file"
  input="variable" input-variable="wd.retrieve.variable"
  format="rfc4180" rootName="Root" rowName="Row" useFirstLineAsHeader="true"/>
```

S1 also checks da.size() greater than zero and at most one. In new code validate availability before indexing where API permits. Adapt delimiter/header/encoding; don't assume RFC4180 defaults parse pipe-separated records. No embedded production fallback rows or unescaped XML concatenation.

### XSLT resources / endpoints / RaaS

```xml
<cc:xslt-plus id="TransformRequest" url="ExampleRequest.xsl"/>
```

Place inside supported steps. Verify actual path/case/suffix/packaging/refresh/message-variable context/processor. Root-level sample doesn't prohibit supported subfolders. Don't alter transformations for a diagram-only fault.

Historical RaaS pattern: exact alias in intsys.reportService.getExtrapath; reject null before transport; real Prompt XML Alias and encoded values. S1 doesn't exercise report API; verify target runtime.

Use actual local-in conventions and documented wcc APIs. S1 audit path uses `vm://wcc/GetEventDocuments`; S3's differently named endpoint needs verification. Similar English names aren't equivalent APIs.

---

## 7. Diagram patterns and reference rules

### Root / ordinary entries

Use section 5's generated envelope. Namespace must match exact sample vocabulary.

```xml
<visualProperties x="52" y="71">
  <element href="assembly.xml#StartHere"/>
</visualProperties>
<connections type="routesTo">
  <source href="assembly.xml#StartHere"/>
  <target href="assembly.xml#Initialize"/>
</connections>
```

S2 has some visualProperties without x/y; defaults/grouping can affect layout. Missing coordinates aren't automatically invalid.

### Global handler: sample-only structural index

```xml
<visualProperties>
  <element href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
</visualProperties>
<connections type="routesTo">
  <source href="assembly.xml#//@beans/@mixed.1/@mixed.39"/>
  <target href="assembly.xml#Note-Error"/>
</connections>
```

This exact example belongs to INT0016. **Do not paste mixed.39 into INT0099.** Current-model Studio/EMF-generated fragment required. XML tag/sibling/line counts aren't FeatureMap indices. Renaming the real handler isn't a URI repair. Ignore malformed historical memory fragments with an inserted scheme after the hash; correct sample begins `#//@beans/`.

Permanent repair PENDING: generate/verify current handler URI, use consistently in visual/edge/membership, test save/reopen and runtime error path.

Temporary workaround APPLIED: remove only broken handler visualProperties, outgoing connection and membership. Keep actual cc:send-error and Note-Error routing. Track missing icon/arrow.

### Nested sources

```xml
<connections type="routesTo">
  <source href="assembly.xml#//@beans/@mixed.1/@mixed.3/@mixed.3"/>
  <target href="assembly.xml#CallCriticalErr1"/>
</connections>
<connections type="routesTo">
  <source href="assembly.xml#//@beans/@mixed.1/@splitter.0/@subRoute.0"/>
  <target href="assembly.xml#FinalizeWS"/>
</connections>
```

Sample-specific indices/targets only. Guessed parent-splitter arrows or invented type=error aren't proven equivalent to nested semantic sources.

### Swimlanes

```xml
<swimlanes x="31" y="9" font="Arial|14|1" fontColor="1"
           name="INT016-Main" labelAlignment="LEFT">
  <elements href="assembly.xml#StartHere"/>
  <elements href="#//@swimlanes.3"/>
  <elements href="assembly.xml#GetIntMap"/>
</swimlanes>
```

Not a transferable stand-alone lane: index depends on original order. S2 demonstrates x/y/name/orientation/font/fontColor/topBorderColor/labelAlignment, not exhaustive schema enumeration.

Do not add width in this model. Height/fgColor omitted for sample alignment unless verified. Preserve supported fonts/colors; old MD's color descriptions were inconsistent. Recompute nested indices after reorder; check cycles/missing targets/unintended duplicate memberships/rogue containers.

### Semantic consistency replaces Triad Rule

- Every used href resolves to intended object, or deliberate omission is documented.
- Connections match real routes and correct nested handlers/branch ports.
- Internal steps need not be independent nodes; terminal nodes need no invented outgoing arrows.
- Lane hierarchy is correct/acyclic; artwork and routing semantics agree.
- Diagram-only change leaves logic untouched; workaround restoration tracked.

Preserve headers/spacing and Group 4's three-column layout where established. INT149C's historical 21-lane hierarchy/dedicated report rows are project-specific, not universal. Pixel spacing is preference, not schema validation; inspect rendered diagram.

---

## 8. XSLT patterns and error prevention

No actual applicant stylesheet supplied. Examples illustrative, not compiled here. Use section 5 header generator and actual input/WSDL; don't invent required request fields or ID types.

### Extension function / QName

Historical stable-project binding:

```xml
xmlns:is="java:com.workday.esb.intsys.xpath.ParsedIntegrationSystemFunctions"
```

```xml
<xsl:variable name="mappedCompany"
              select="is:getIntegrationMapValue('Company', $rawCompany)"/>
```

Verify processor/runtime support. Offline/browser/.NET execution without Workday extension support doesn't prove mapping works. exclude-result-prefixes controls namespace output, not implementation.

Invalid parameter names:

```xml
<xsl:param name="inmap.Bank Account"/>
<xsl:param name="inmap.Ledger Account"/>
```

Spaces violate QName syntax; period isn't the issue. Use valid parameter names with explicit passing, or stable project's inline map-name string. Strings may contain spaces. Preserve map name/direction/key.

### Mapping and whitespace

Historical Ad Hoc Bank rule: raw CSV Company → Company map; use nonempty/different mapped result else raw. Bank Account lookup uses raw Company, optional resolved-company fallback only where approved.

```xml
<xsl:variable name="resolvedCompany"
  select="if (normalize-space(string($mappedCompany)) != ''
              and string($mappedCompany) != $rawCompany)
          then string($mappedCompany) else $rawCompany"/>
```

Declare rawCompany from actual source node/namespace, not invented alias.

```xml
<wd:ID wd:type="Company_Reference_ID"><xsl:value-of select="normalize-space($resolvedCompany)"/></wd:ID>
```

Use actual service ID type. normalize-space collapses internal whitespace too; don't indiscriminately apply to addresses/memos. Inspect emitted payload rather than blaming source indentation.

### Input context / amounts / dates / SOAP

Bind real namespaces; distinguish full SOAP/message/variable/split-record input. Test empty/multiple matches. Validate numbers before arithmetic (no silent NaN), actual separators/currency, decimal precision and sign. Parse known date formats and real dates; don't guess ambiguous day/month. Maintain approved fallback precedence.

Historical Ad Hoc Bank specifics only: negative → withdrawal, nonnegative → deposit; absolute transaction/line magnitudes; USD/ledger 12020 defaults; validated DD-MM-YYYY or DD/MM/YYYY → YYYY-MM-DD.

Old MD's fixed 0.01 tax multiplier, descriptor currency-symbol fallback and negative reversal formula aren't universal financial logic. Verify rate/source/currency/sign/requirements.

Use XML-aware serialization, not unescaped concatenation; test &, angle brackets, quotes, Unicode, blanks and delimiters. Don't use disable-output-escaping as generic fix. Avoid duplicate SOAP wrapping; S1 wraps in one payroll path, not every endpoint contract. Align application/version/schema; inspect SOAP/application faults, not HTTP alone. Separate payload correctness from target lifecycle/security.

### XSLT error register

| Symptom | Fix/check | Status |
|---|---|---|
| Invalid QName | Valid parameter name or verified inline mapping string | Historical S5/S7 |
| Unknown extension | Actual binding and supported processor/runtime | Recommended diagnostic |
| Stylesheet not found | Path/case/package/refresh | Historical; no blanket subfolder ban |
| Empty mapped ID | Exact map name/key/direction/source namespace/fallback | Historical pattern + tests |
| Empty XPath | Actual root/namespace/message context | General diagnostic |
| NaN/invalid amount | Validate/cast and approved precision/sign | General diagnostic |
| Invalid date | Known format and real-date validation | General diagnostic |
| PRNUM/PRLINENUM wrong | Verified INT0047 logic in both branches | Historical S7 |
| Target rejects valid payload | Inspect lifecycle/security separately | Historical Maximo case |

---

## 9. Project-specific historical lessons

These aren't current tenant verification or universal configuration.

### INT0099 remaining review findings (S3)

- Existing-email logic resolves path but doesn't demonstrate real fetch/full parsing; hardcoded email instead.
- Missing file can become embedded applicant rows; sequence failure becomes 1.
- Mixed-language expressions need compilation; CSV concatenation lacks reliable XML escaping.
- Success.Count can include New records despite validation-only blocking load; separate validated/loaded.
- SOAP response/confirmed successful creation before counting not demonstrated.
- Duplicate normalization and retry-after-failure need tests.
- Retention attribute exists but actual summary retention use not demonstrated.
- Endpoints/processor/SOAP version/package resources need verification.

Code-review findings, not newly reproduced runtime failures. Diagram loading isn't production readiness.

### INT149C (historical S5)

Exact report alias, actual Prompt XML Alias/View URLs, correct real-time/delta contract, effective ISU report/data-source/domain/field access, caught-error counters and final-message gating. Preserve report rows/group hierarchy. Current tenant settings not retrieved.

### INT0076 EMWD8 after refresh (historical S7)

Previously resolved: pending auth policy unactivated; intended External ISU Access needed correct ISSG and User Name Password authentication; intended ISU membership and tenant-qualified username mattered. Activate authorized pending changes, verify intended endpoint/tenant (not accidental SBX), retest event and end-to-end inbound. Don't copy another project's ISU/ISSG or broaden policy generically. No credentials stored here.

### INT0047 mapping versus Maximo history (historical S7)

Stable `xslt/WSRequestFormat.xsl`; update Change and AddChange branches. Verified Work Order flag true → suppress PRLINENUM/PRNUM; otherwise line number from wd:lineNum, PRNUM from wd:reqLnRef/wd:ID[@wd:type='Requisition_Reference_ID']. Verify alias CF_INT0047_is_Work_Order_exists__ and true representation in raw CR XML. No assembly/Integration Attribute changes required for that mapping.

Historical PO-10000031 payload had empty values on both lines. Separate BMXAA4137E cause: STATUS=CLOSE, HISTORYFLAG=1, STATUSIFACE=1. Use populated PO prompt and eligible non-closed PO; history guard only if required. Don't turn Work Order logic into closed-PO logic or claim mapping bypasses lifecycle restriction.

### Ad Hoc Bank (historical S7)

Stable XSLT 2.0, established inline map lookups, no parameter names with spaces, raw/resolved Company roles preserved, established fallback fields, project-specific USD/12020/sign behavior.

### Not reconstructed

INT0045/INT0046 goals are context, but current source/error logs weren't supplied; no fixes invented or marked tested for them.

---

## 10. Safe local diagnostics and repair procedures

### Usage

Windows+R → powershell → Enter. Normally no admin needed. Copy full block, not fences/PS prompt. Ctrl+C cancels unfinished input. Pasting reviewed commands needs no execution-policy change; don't bypass organizational security.

Scripts here haven't been executed by the assistant. Earlier user-run patches are recorded separately.

### Read-only audit

```powershell
& {
    $ErrorActionPreference = 'Stop'
    $dir = (Read-Host 'Paste full WSAR-INF folder path').Trim().Trim('"')
    foreach ($name in @('assembly.xml','assembly-diagram.xml')) {
        $path = Join-Path $dir $name
        Write-Output "`n=== $name ==="
        if (!(Test-Path -LiteralPath $path)) {
            Write-Output "MISSING: $path"
            continue
        }
        $text = [System.IO.File]::ReadAllText($path)
        Write-Output "Path: $path"
        Write-Output "SHA256: $((Get-FileHash -LiteralPath $path -Algorithm SHA256).Hash)"
        Write-Output "Redaction marker: $($text.Contains('<redacted URL>'))"
        try {
            [xml]$xml = $text
            Write-Output "Root: $($xml.DocumentElement.Name)"
            Write-Output "Namespace: $($xml.DocumentElement.NamespaceURI)"
            if ($name -eq 'assembly-diagram.xml') {
                $ref = $xml.SelectSingleNode("/*/*[local-name()='element']")
                if ($null -ne $ref) {
                    Write-Output "Assembly reference: $($ref.GetAttribute('href'))"
                }
                $attrs = $xml.SelectNodes("//*[local-name()='swimlanes']/@*[name()='width' or name()='height' or name()='fgColor']")
                Write-Output "Swimlane attrs to review: $($attrs.Count)"
                $bad = $xml.SelectNodes("//@href[.='assembly.xml#GlobalErrorHandler']")
                Write-Output "Known broken handler-ID uses: $($bad.Count)"
            }
        } catch {
            Write-Output "XML parse error: $($_.Exception.Message)"
        }
    }
}
```

Does not write, resolve EMF, validate XSD or compile XSLT.

### Narrow namespace/swimlane repair example

Audit first, close Studio, select one mode and test one layer. Only for sample-compatible shape: unprefixed swimlanes/double-quoted attributes. No arbitrary reference repair or business logic changes.

```powershell
& {
    $ErrorActionPreference = 'Stop'
    function New-NotationNamespace {
        param([bool]$OldFormat)
        $slash = [string][char]47
        $hostLabels = @('workday','com')
        $segments = @('studio','editors','notation')
        if ($OldFormat) {
            $hostLabels = @('www','workday','com')
            $segments += '1.0'
        }
        return ('http' + ':' + $slash + $slash +
                ($hostLabels -join '.') + $slash + ($segments -join $slash))
    }
    $mode = Read-Host 'Enter Namespace or SwimlaneAttributes'
    if ($mode -notin @('Namespace','SwimlaneAttributes')) {
        throw 'Unknown mode. No changes made.'
    }
    $dir = (Read-Host 'Paste full WSAR-INF folder path').Trim().Trim('"')
    $assembly = Join-Path $dir 'assembly.xml'
    $diagram = Join-Path $dir 'assembly-diagram.xml'
    $hash = (Get-FileHash -LiteralPath $assembly -Algorithm SHA256).Hash
    $text = [System.IO.File]::ReadAllText($diagram)
    [xml]$before = $text
    $updated = $text
    $expected = New-NotationNamespace -OldFormat $false
    if ($mode -eq 'Namespace') {
        $old = New-NotationNamespace -OldFormat $true
        if ($before.DocumentElement.NamespaceURI -eq $expected) {
            Write-Host 'Already corrected. No files changed.'
            return
        }
        if ($before.DocumentElement.NamespaceURI -ne $old) {
            throw 'Unexpected namespace. No changes made.'
        }
        $oldDecl = 'xmlns:wdnm="' + $old + '"'
        $newDecl = 'xmlns:wdnm="' + $expected + '"'
        if (!$text.Contains($oldDecl)) {
            throw 'Unexpected declaration syntax. No changes made.'
        }
        $updated = $text.Replace($oldDecl,$newDecl)
    }
    if ($mode -eq 'SwimlaneAttributes') {
        if ($before.DocumentElement.NamespaceURI -ne $expected) {
            throw 'Namespace differs from sample. Inspect first.'
        }
        $updated = [regex]::Replace(
            $text, '<swimlanes\b[^>]*>',
            [System.Text.RegularExpressions.MatchEvaluator] {
                param($match)
                return [regex]::Replace($match.Value,
                    '\s+(?:width|height|fgColor)\s*=\s*"[^"]*"', '')
            }
        )
    }
    [xml]$candidate = $updated
    if ($candidate.DocumentElement.NamespaceURI -ne $expected) {
        throw 'Candidate namespace invalid. No changes made.'
    }
    if ($mode -eq 'SwimlaneAttributes') {
        $remaining = $candidate.SelectNodes("//*[local-name()='swimlanes']/@*[name()='width' or name()='height' or name()='fgColor']")
        if ($remaining.Count -gt 0) {
            throw 'Targeted attrs remain in unexpected format. No changes made.'
        }
    }
    if ($updated -eq $text) {
        Write-Host 'No targeted edit required. No files changed.'
        return
    }
    $backup = Join-Path $env:LOCALAPPDATA ('WorkdayStudio-Backups\' + (Get-Date -Format 'yyyyMMdd-HHmmss-fff'))
    New-Item -ItemType Directory -Path $backup | Out-Null
    Copy-Item -LiteralPath $assembly -Destination $backup
    Copy-Item -LiteralPath $diagram -Destination $backup
    $utf8 = New-Object System.Text.UTF8Encoding($false)
    [System.IO.File]::WriteAllText($diagram,$updated,$utf8)
    [xml]$saved = [System.IO.File]::ReadAllText($diagram)
    if ((Get-FileHash -LiteralPath $assembly -Algorithm SHA256).Hash -ne $hash) {
        throw 'Assembly changed concurrently. Stop and compare backup.'
    }
    Write-Host "Backup: $backup"
    Write-Host "Applied: $mode"
    Write-Host "Saved namespace: $($saved.DocumentElement.NamespaceURI)"
    Write-Host 'Assembly unchanged; Studio testing still required.'
}
```

Limits: narrow text patch, UTF-8 rewrite, not atomic or EMF validator. Inspect original encoding and diff. Don't rerun already-applied modes unnecessarily.

### Handler proxy workaround boundary

Capture exact URI; confirm real handler and target; prefer generated fragment. If visual workaround accepted: backup, remove exact visual/edge/membership, prove assembly unchanged, record lost artwork, retest. No generic delete-all-errors regex.

### Refresh and rollback

Start Studio → correct project → F5 → reopen. Project → Clean helps build/resource checking, not invalid notation repair. If opens, save/close/reopen. Inspect newest relevant Error Log/Problems, not unrelated proxy/update warnings.

Rollback: close Studio, preserve current files, identify correct backup stage, restore only intended files, refresh/retest. Don't automatically reintroduce wrong namespace to hide a later error.

No workspace metadata deletion, all-lane/all-edge removal, reinstall or proxy/security change without evidence/recovery plan. Automatic notation regeneration on deletion wasn't verified for this build.

---

## 11. Validation and release checklist

### Levels

| Level | Check | Establishes |
|---|---|---|
| V1 | XML/XSL parsing | Well-formedness only |
| V2 | Installed XSD/metamodel | Supported version-specific structure |
| V3 | Current model reference resolution | Correct nonproxy targets for tested references |
| V4 | Diagram open/save/close/reopen | Editor load/serialization/layout |
| V5 | Expressions/XSLT/resources | Actual processor compilation and resource presence |
| V6 | Test build/package/deploy | Works in that environment |
| V7 | Controlled end-to-end execution | Tested outputs/target updates/error behavior |

Never report V7 after only V1. Saving a file proves no validation level automatically.

### Pre-development

- [ ] Current assembly/diagram/XSL/requirements and matching sample.
- [ ] Studio/Java build, target tenant and service/report versions.
- [ ] Actual Resource → Location; no wrong same-name copy.
- [ ] Baseline backup/hash/diff scope.
- [ ] INT0016 reference pair retained together and reviewed using section 14; new project identifiers and resources mapped explicitly.

### Assembly

- [ ] Exact locally expanded namespaces; no redaction/Markdown in values.
- [ ] Supported version and service order/configuration.
- [ ] Exact parameter/attribute/sequencer/alias names.
- [ ] Supported strategies/branches/splitter and valid route targets.
- [ ] Existing resources, correct message/variable input/output.
- [ ] Verified endpoints/APIs and actual target identifiers.
- [ ] No fake production fallback; correct counters/continuation/messages.
- [ ] Validated versus actually loaded records distinguished.

### Diagram

- [ ] Correct notation and supported swimlane attributes.
- [ ] Intended model types resolve, current fragments not copied indices.
- [ ] Correct nested branch/handler sources and lane indices.
- [ ] No cycles/rogue containers; headers/spacing/columns preserved.
- [ ] Workaround omissions/restoration tracked.
- [ ] Diagram-only diff leaves assembly/XSL unchanged.
- [ ] Open/save/close/reopen succeeds.

### XSLT

- [ ] Actual stylesheets reviewed, not guessed from filenames.
- [ ] Processor/version/extensions supported.
- [ ] Valid QNames and exact mapping names/direction/keys.
- [ ] Real namespace/context, empty/multiple and fallback cases.
- [ ] Amount/date/sign/currency/defaults approved and tested.
- [ ] Special characters/Unicode safe.
- [ ] Correct WSDL/ID types/application/version/SOAP handling.
- [ ] Change/AddChange or equivalent branches covered.

### Runtime cases

Missing/empty/multiple files; wrong delimiter/header/encoding; valid/missing/invalid fields; duplicates/case/whitespace; Unicode; batching; validation-only no writes; missing/unauthorized/empty report and wrong prompts; missing sequence/map/config; timeout/SOAP/business fault; retries/partial success; early initialization failure; correct counts/messages/summary/retention; actual target state; eligible objects/populated prompts.

### Standard delivery record

```text
Files read / changed:
Exact error / reproduction:
Evidence/source:
Patch scope/diff:
Backup/rollback:
XML parsing:
Schema/metamodel:
Reference resolution:
Editor open/save/reopen:
Expression/XSLT compilation:
Build/deploy:
Runtime tests:
Remaining risks:
```

Use not run / unavailable / pending honestly. Confidence isn't a test result.

---

## 12. Web / Workday Community research

### Recorded research for version 1.0

Date: 2026-11-13. The entries below are retained from the uploaded version 1.0 guide. No new web or Community research was performed for the version 1.1 reference-file update.

- Searched available enterprise sources for Studio GMFReader/None.get, notation namespace, unsupported width, GlobalErrorHandler proxy/MatchError, xslt-plus and MVEL; narrowed exact-term follow-up.
- Restricted by user-selected sources to **microsoft_sharepoint_sl, help_aggregator, jira**. This is source selection, not proof of missing permissions.
- No authoritative matching Workday Studio fix verified from returned results.
- Direct public XML namespace/XSLT/EMF documentation reads were blocked because URLs couldn't be verified against trusted sources. No content retrieved or retry made.
- No live public-web search or authenticated Workday Community search available in that research session.
- No matching Community solution, release-specific known issue or Workday bug ID verified.

**Repairs here derive from uploaded samples and local user-confirmed results, not a verified Community post.**

### Rejected false leads

- [Invalid XML Character Causing Workflow Page not able to be viewed](https://support.atlassian.com/jira/kb/invalid-xml-character-causing-workflow-page-not-able-to-be-viewed/) — Jira workflow storage, not Studio.
- [How to identify Workflow Diagram in Jira database](https://support.atlassian.com/jira/kb/how-to-identify-workflow-diagram-in-jira-database/) — unrelated diagram format.
- [Error converting XML to workflow descriptor when Upgrading JIRA](https://support.atlassian.com/jira/kb/error-converting-xml-to-workflow-descriptor-when-upgrading-jira/) — different product/schema.
- [How to Fix Entity Name Must Immediately Follow '&' Error in Jira Workflow Scheme](https://support.atlassian.com/jira/kb/jira-workflow-scheme-entity-name-error/) — not a Studio migration.
- Confluence Team Calendars/xwork descriptor results and Workday business/reference-ID conversion tickets — not GMFReader/EMF repair evidence.

Do not apply Jira SQL repairs/plugin deletion/unrelated caches to Studio just because exception words overlap.

### Public reference reads attempted — NOT retrieved

Listed as host labels and path components to avoid automatic redaction; scheme HTTPS. Research targets, not supporting citations:

- Host www, w3, org; path TR, REC-xml-names, trailing empty segment.
- Host www, w3, org; path TR, xslt20, trailing empty segment.
- Host download, eclipse, org; path modeling, emf, emf, javadoc, 2.11, org, eclipse, emf, ecore, resource, Resource.html.

All three declined on trusted-source verification grounds.

### Queries for future live web / Community access

```text
"Workday Studio" "GMFReader.scala" "None.get"
"Workday Studio" "GmfElement" "Diagram" "NotationFileMigrator"
"Workday Studio" "Feature 'width' not found"
"Workday Studio" "assembly-diagram.xml" "swimlanes"
"Workday Studio" "scala.MatchError" "eProxyURI"
"Workday Studio" "global-error-handler" "mixed"
"Workday Studio" "execute-steps-when" "async-mediation"
"Workday Studio" "sequence-generator-service" "retrieval-service"
"Workday Studio" "getSequencedValue"
"Workday Studio" "xslt-plus" "getIntegrationMapValue"
```

Add actual Studio build. No credentials/tokens/personal records in public searches/posts.

### Before adopting a searched fix

Open actual source; record title/URL/publisher/date; confirm Workday Studio not adjacent product; match error/build/format; distinguish official/community/speculation; test isolated copy with backup; add results/applicability here.

```markdown
### Research item: <title>
- Retrieved date:
- Actual URL:
- Publisher/author:
- Product/version:
- Exact error matched:
- Cause and proposed fix:
- Authority/relevance limits:
- Local test result:
- Adoption: pending / rejected / approved for named version
```

Support package: actual Studio/Java build, post-patch XML pair, same-build sample, newest exact error/time/URI/line-column, reproduction/location/diffs, whether sample opens, relevant Problems/Error Log. Remove secrets/unneeded personal data; don't submit stale files as patched baseline.

---

## 13. Future prompt, change log and open work

Attach this MD with current files and the INT0016 reference pair in future chats. File creation doesn't automatically make it accessible in every later conversation. Refer to section 14 for source links, local filenames and reuse rules.

### Reusable prompt

```text
Use WORKDAY_STUDIO_DEVELOPMENT_KNOWLEDGE_BASE.md with actual current
files and installed Studio schemas/metamodel.
Task: [new integration / change / Sync diagram to assembly / fix error]
Project/integration:
Studio build:
Target environment:
Requirements:
Attachments:
- WORKDAY_STUDIO_DEVELOPMENT_KNOWLEDGE_BASE.md
- INT0016_sample_assembly.txt
- INT0016_sample_assembly-diagram.txt
- Current assembly.xml and assembly-diagram.xml, when changing a project
- Actual relevant XSL/XSLT files and input/output examples

Use the two INT0016 sample files together as the structural reference
pair described in section 14. Read them before designing a new project.
They contain XML despite their .txt extensions. Do not copy tenant WIDs,
service names, endpoints, stylesheet names, payroll business logic or
EMF indices into the new project unchanged. Use a closer same-build
known-good project if requirements or target model need it.

Read all files. Preserve logic/service names/group headers/hierarchy
unless explicitly changed; preserve Group 4 three-column layout where
established. Don't invent namespaces/attributes/APIs/endpoints/request
schemas/EMF indices. XML id isn't automatically EMF ID. Resolve actual
handler/branch objects. Keep diagram semantics aligned with assembly.
Use local namespace generation if delivery redacts URLs.

Back up; patch one layer at a time. No silent production fake-data or
constant-sequence fallback. Deliver diffs, validation results, pending
checks and rollback. Update this MD for new evidence/fixes/tests.
Never claim zero-error or runtime testing without execution.

For research verify actual web/Community sources and version match;
cite real URLs. Record unavailable/unrelated results honestly; don't
invent a verified Community fix.
```

### Version 1.0 change log

Consolidated recorded errors/sample patterns/local outputs; documented namespace/width/proxy progression; separated temporary workaround from permanent repair; added assembly/XSLT rules and project history; retracted fixed indices/Triad/blanket bans; added diagnostics/tests/research limits; replaced sanitizer-vulnerable namespace examples with local component-based generation.

### Version 1.1 change log — 2026-11-13

- Read the uploaded version 1.0 MD and both latest INT0016 sample files.
- Registered the latest sample uploads as S1/S2, preserving earlier links for traceability.
- Added section 14: matched reference manifest, portable folder layout, sample-pattern inventory, project adaptation rules and future-development workflow.
- Updated the reusable prompt and pre-development checklist to explicitly require both sample files.
- Distinguished sample inspection from same-build Studio validation and actual runtime testing.
- Retained existing error/fix registers, namespace-generation examples, local repair procedures, XSLT guidance and open issues.
- Added a caution that S1 accesses the retrieved document before its size checks; reuse the pattern thoughtfully, not every statement order blindly.
- Recorded that no new web/Community research or project XML changes were performed for this documentation update.

### Open issues

- [ ] Studio build supplied.
- [ ] Final diagram open/save/close/reopen confirmed.
- [ ] Handler artwork restored via current-model fragment if required.
- [ ] Nested handlers/branches verified and post-patch baseline retained.
- [ ] Actual applicant XSL reviewed; INT0099 implementation risks resolved.
- [ ] Service order/processors/endpoints checked against target runtime.
- [ ] Build/package/deploy and controlled runtime tests completed.
- [ ] Matching official Workday/Community evidence added when accessible.
- [ ] Local/reference-repository copies of both INT0016 samples retained alongside this MD; upload links alone are not the archive.

**Maintenance:** version known-good files. Never replace working baseline with untested output. Append evidence-backed changes whenever assembly, notation or XSLT changes.

---

## 14. INT0016 matched reference pair for future Studio projects

### 14.1 Reference manifest

**Use both files together when creating a new Studio project or synchronizing its diagram.** These are the user-provided INT0016 samples, not newly generated project files.

| Reference file | Role | Latest uploaded source |
|---|---|---|
| `INT0016_sample_assembly.txt` | XML assembly reference: component structure, launch parameters, services, expressions, routing, retrieval, transformations, SOAP calls, audit processing and error handling | [Open sample assembly](file://user/fileupload/461cd97c-e42c-4b4f-a8b7-f2639e92b348) |
| `INT0016_sample_assembly-diagram.txt` | XML notation reference: root declaration, visual properties, connections, structural EMF fragments, group hierarchy and nested swimlanes | [Open sample diagram](file://user/fileupload/d770617c-4244-47c2-bdde-2f0f7fe3a802) |

Both files were read through their closing tags for this update. The sample assembly declares version `2022.10` and contains main and audit entry points named `INT016 TBX Benefit Deductions Inbound` and `INT016 TBX Benefit Deductions Inbound Audit`. The uploaded filenames use `INT0016`; internal names use `INT016`. Preserve original sample files as supplied, and choose new integration names deliberately rather than applying an uncontrolled global replacement.

**Evidence limit:** these files were supplied as the working reference. Reading them does not independently prove they open, compile or run on a different installed Studio build. Record the build and editor/runtime validation before promoting any adapted project to a known-good baseline.

### 14.2 Keep a portable reference folder

Suggested local or version-controlled layout:

```text
Workday-Studio-Knowledge/
  WORKDAY_STUDIO_DEVELOPMENT_KNOWLEDGE_BASE.md
  references/
    INT0016/
      INT0016_sample_assembly.txt
      INT0016_sample_assembly-diagram.txt
  projects/
    <new-project-name>/
      ws/
        WSAR-INF/
          assembly.xml
          assembly-diagram.xml
          <actual-stylesheet-files>.xsl
```

If you use that layout, these relative links point to the retained originals:

- [Local reference assembly](references/INT0016/INT0016_sample_assembly.txt)
- [Local reference diagram](references/INT0016/INT0016_sample_assembly-diagram.txt)

The relative links work only after you place the files in those folders. This MD update does not create those folders on your computer, bundle the sample files or embed their full contents. Keep unmodified originals; develop in separate project files. Do not rely solely on chat upload URLs, which may not be accessible in later sessions.

### 14.3 What to reference in assembly.xml

The sample demonstrates:

- Spring/Cape Clear/Workday namespace declarations; construct exact values using section 5 if delivery redacts URLs.
- Assembly `id="WorkdayAssembly"` and sample version `2022.10`.
- Main and audit `cc:workday-in` entry points.
- Launch parameter definitions, attribute-map services, a deduction integration map, sequence-generator services, retrieval service and service-reference declarations.
- `lp.getSimpleData(...)`, `lp.getReferenceData(...)`, `lp.getSequencedValue(...)` and `intsys.getAttribute(...)` in their shown contexts.
- `cc:async-mediation` with `cc:steps`, individual expressions, validation and nested error handling.
- Conditional `execute-when` on `cc:local-out` and `execute-steps-when` on `cc:async-mediation`.
- Local calls and response routing; `routes-to` and `routes-response-to` have different semantics.
- Retrieved-document processing, CSV-to-XML conversion and empty-file handling.
- `cc:xslt-plus` with message or variable contexts and actual stylesheet URLs.
- `cc:route` with `cc:all-strategy`, named sub-routes and a document iterator where shown.
- Splitters with named sub-routes and `cc:xml-stream-splitter`.
- Aggregation, audit ZIP/storage, retention expressions and integration messages.
- Paged Workday calls, SOAP wrapping and a Payroll SOAP output component.
- Global and nested error handlers, critical-message routes and continuation flags.

These are **observed patterns**, not a full schema/API catalogue. Select the patterns required by the new project's requirements. A new applicant, Maximo, worker or bank integration does not automatically require the sample's payroll, audit, splitter or aggregator flow.

**Do not copy statement order without reviewing safety:** in `GetInputFile`, the sample invokes `da.toVar(0,...)` before validating `da.size()`. For new development, verify document availability before indexing where the supported API permits; test zero, one and multiple documents. This is a preventive review finding, not a newly reproduced failure in the sample.

### 14.4 What to reference in assembly-diagram.xml

Use the sample for notation vocabulary and layout patterns:

- Exact notation namespace: HTTP scheme, host labels `workday`, `com`, path segments `studio`, `editors`, `notation`; no extra `www` or `1.0`. Use section 5's generator for the literal declaration.
- Root model reference: `assembly.xml#WorkdayAssembly` for that actual assembly ID.
- `visualProperties` entries, some with explicit coordinates and others using defaults/group placement.
- `connections` with `type="routesTo"` or `type="routesResponseTo"`, source/target references, optional anchors and bendpoints.
- Horizontal functional groups and vertical nested swimlanes.
- Supported sample attributes such as `x`, `y`, `name`, `orientation`, `font`, `fontColor`, `topBorderColor` and `labelAlignment` where shown.
- Structural references for the sample's global handler and nested error/route/splitter objects.
- Local nested lane references such as `#//@swimlanes.N`, whose indices depend on the current diagram order.

Do not add `width` to this notation model: it was explicitly rejected in the user's incident. `height` and `fgColor` were also removed for sample alignment; their universal invalidity across every Studio version was not established.

The sample's `@mixed.39` handler URI is **reference syntax only**, not a reusable index. Generate or verify the fragment for the new assembly model. Inspect actual error/branch object types and lane membership rather than assuming that every XML ID can be referenced as `#ID`.

### 14.5 Mandatory project-adaptation rules

1. Read the current requirements, this guide and both sample files before generating a new pair.
2. Record Studio/Java build and target environment. Prefer a more directly matching same-build known-good example if needed.
3. Create a project-specific naming/configuration map: integration/entry-point names, top-level component IDs, parameters, attributes, service names, sequence names, local endpoints, maps, resource filenames and actual target identifiers.
4. Do not carry over sample tenant WIDs, field-override references, payroll APIs, report aliases or business rules without a verified requirement.
5. Do not mechanically replace every `id` occurrence: nested step IDs can have different scope, and endpoint/resource strings need separate review.
6. Build/validate the new assembly first, then generate or synchronize the diagram against that assembly. Preserve the user's existing group headers/hierarchy and three-column Group 4 layout where applicable.
7. Resolve all model references in the actual new model. Never transplant `@mixed.N`, `@route.N`, `@splitter.N`, `@subRoute.N` or lane indices between unrelated files.
8. Retain genuine error-handling logic. Removing broken artwork is a documented workaround only, not the permanent default for new projects.
9. Obtain actual referenced XSLT resources. These two samples contain stylesheet calls, not the stylesheet implementations; no inferred XSLT contents are supplied by this reference pair.
10. Do not infer universal bans from omissions in the sample: comments, subfolders, processors and language constructs require their own target-version validation.
11. Validate XML, target schema/metamodel, model references, editor open/save/reopen, resources, build and runtime separately using section 11.
12. Record evidence, test results, remaining limitations and any new errors/fixes in this MD. A generated or saved file is not automatically a tested integration.

### 14.6 New-project workflow

```text
Read requirements + MD + both INT0016 reference files
    ↓
Confirm target Studio build, schemas and required resources
    ↓
Map project-specific names, APIs, input/output and business rules
    ↓
Create/validate assembly.xml using applicable sample patterns
    ↓
Create/sync assembly-diagram.xml against the actual new model
    ↓
Add/review real XSLT resources and transformation tests
    ↓
Validate XML → schema → EMF references → editor → build → runtime
    ↓
Update this MD and archive the tested project/reference baseline
```

For existing-project changes, read the actual latest files before altering anything; do not replace a known-good pair wholesale with this sample.

### 14.7 Short future request

```text
Use the attached WORKDAY_STUDIO_DEVELOPMENT_KNOWLEDGE_BASE.md,
INT0016_sample_assembly.txt and INT0016_sample_assembly-diagram.txt
as the reference set for this new Workday Studio project.

Project name:
Studio build:
Integration requirements:
Input/output examples:
Required services and XSLT:
Layout requirements:

Adapt the reference patterns, not its project-specific identifiers or
EMF indices. Create consistent assembly and diagram files, review the
actual XSLT, state validation performed versus pending, and update the
knowledge base with evidence-backed errors, fixes and test results.
```

When asked **“Sync diagram to assembly”**, read the current assembly, update only the intended notation/layout, preserve established group headers/hierarchy (including the Group 4 three-column arrangement where present), and document reference validation and layout changes. Do not silently modify integration business logic.
