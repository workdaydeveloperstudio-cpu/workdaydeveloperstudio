## INT0016 reference files for future Workday Studio development

### Reference pair

Keep these original files alongside this knowledge base:

- INT0016_sample_assembly.txt
- INT0016_sample_assembly-diagram.txt

Their contents are XML despite the .txt extensions. Use them together
as reference patterns—not as universally valid, drop-in project files.

### assembly.xml reference patterns

The sample declares assembly version 2022.10 and demonstrates:

- Launch parameters and integration attributes.
- Sequence-generator and retrieval-service declarations.
- lp.getSimpleData(...), lp.getSequencedValue(...), and
  intsys.getAttribute(...).
- Retrieval validation and CSV-to-XML conversion.
- cc:xslt-plus transformation steps.
- Routes, named sub-routes, XML stream splitters and aggregators.
- Local calls, SOAP calls, audit processing and error handling.
- execute-when on local-out and execute-steps-when on async-mediation.

These are observed sample patterns, not a complete schema specification.
Validate compatibility with the target Studio build.

### assembly-diagram.xml reference patterns

The sample uses:

<wdnm:Diagram xmlns:wdnm="http://workday.com/studio/editors/notation">

Its root model reference is:

<element href="assembly.xml#WorkdayAssembly"/>

It demonstrates visualProperties, connections, horizontal groups,
vertical nested swimlanes and structural EMF references.

### Mandatory adaptation rules

1. Read the current assembly before creating or synchronizing its diagram.
2. Adapt integration names, component IDs, service names, endpoints,
   attributes, launch parameters and resource paths.
3. Do not copy INT0016's @mixed.N, @route.N, @splitter.N or swimlane
   indices into a different assembly without resolving them.
4. An XML id does not guarantee that an EMF reference resolves.
   The sample global handler uses a structural fragment.
5. Preserve actual error-handling logic when repairing diagram references.
   Removing a broken icon is a workaround, not a permanent reference fix.
6. Use sample-supported swimlane attributes. The sample omits width,
   height and fgColor; do not reintroduce them into the repaired diagram
   without confirming support in the target notation model.
7. Referenced XSLT contents are not included in these two samples.
   Obtain and review those files separately.
8. Do not infer universal restrictions from this sample's omissions:
   comments, XSLT subfolders and expression constructs require their
   own validation.

### Validation checklist

- XML is well-formed and namespace values are intact.
- Assembly passes target Studio schema validation.
- Diagram references resolve without EMF proxies.
- Connections reflect actual routes and response routes.
- Nested swimlane references resolve and groups do not overlap.
- Referenced XSLT resources exist and compile.
- Diagram opens, saves, closes and reopens successfully.
- Integration passes appropriate non-production runtime tests.

### Future-development instruction

“Use this knowledge base and the attached INT0016 reference pair.
Adapt the sample patterns to the current project, preserve required
business logic, validate model references, and document new errors,
fixes and actual test results.”