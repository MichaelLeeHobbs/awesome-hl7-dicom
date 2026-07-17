# Awesome HL7 and DICOM [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of HL7 and DICOM resources, tools, and hard-won field notes.

Healthcare interop is a field where the good tools are hard to find, the bad ones look identical to
the good ones, and half the top search results have been abandoned since 2022. This list tries to be
the thing you'd want a senior integration engineer to hand you on day one.

**The curation bar:** every entry is something a working integration engineer would be glad someone
showed them. Maintenance status is stated honestly — `(unmaintained)` and `(commercial)` are labels,
not insults, and a dormant-but-canonical tool still earns its place. Things that are dead *and*
superseded live in [Dead ends](#dead-ends), because knowing what **not** to adopt saves more time
than one more link.

Last verified: **July 2026**.

## Contents

- [HL7](#hl7)
  - [Notes and Tips](#notes-and-tips)
  - [The Standard](#the-standard)
  - [Integration Engines](#integration-engines)
  - [Engine Plugins and Tooling](#engine-plugins-and-tooling)
  - [Libraries](#libraries)
  - [Tools and Validators](#tools-and-validators)
  - [Test Data](#test-data)
  - [FHIR](#fhir)
  - [Learning](#learning)
- [DICOM](#dicom)
  - [DICOM Notes and Tips](#dicom-notes-and-tips)
  - [The DICOM Standard](#the-dicom-standard)
  - [Toolkits and Libraries](#toolkits-and-libraries)
  - [Servers and Archives](#servers-and-archives)
  - [Viewers](#viewers)
  - [DICOM Tools and Utilities](#dicom-tools-and-utilities)
  - [DICOM Learning](#dicom-learning)
- [Security](#security)
- [Dead ends](#dead-ends)

---

# HL7

## Notes and Tips

Unofficial field notes. These are the things that cost somebody a weekend.

- **Read the spec.** Actually read it. You will learn more in one afternoon with
  [the v2 standard](https://www.hl7.org/implement/standards/product_brief.cfm?product_id=185) than in
  a month of guessing. It's been free since 2020 — there's no excuse left.
- **Expect that the people you're integrating with have not read the spec.** Your parser has to
  survive their interpretation of it. Be liberal in what you accept.
- **Follow the standard even when you can get away with not following it.** You *can* stuff a
  20-character Accession Number into a field — and it will work right up until it reaches a DICOM
  system, where `AccessionNumber` is a hard 16-character limit. The break happens three systems away
  from the mistake, months later.
- **"HL7 v2 is pipe-delimited" is a trap.** The delimiters are declared in `MSH-1`/`MSH-2` and you
  are supposed to read them, not assume them. Also: escape sequences (`\F\`, `\S\`, `\T\`, `\E\`,
  `\R\`) mean a field's raw text is not its value.
- **Z-segments are where the money is and the spec isn't.** Every real interface has them. Document
  yours, because nobody else will document theirs.
- **`MSH-9` tells you the message type; the trigger event tells you what actually happened.** An
  `ADT^A08` that arrives before the `A01` is not a bug in your code — it's Tuesday. Design for
  out-of-order and duplicate messages from day one.
- **ACKs are a protocol, not a formality.** Decide early whether you're doing original or enhanced
  mode acknowledgment, and what an `AE` vs `AR` actually means to your retry logic.
- **MLLP is a two-byte wrapper around a TCP stream and nothing more.** No framing guarantees, no
  built-in TLS, no auth. Treat the transport as hostile.

## The Standard

- [HL7 Standards Index](https://www.hl7.org/implement/standards/) - The front door to everything.
- [HL7 v2 Product Brief](https://www.hl7.org/implement/standards/product_brief.cfm?product_id=185) -
  The workhorse. ~95% of real-world clinical messaging is still v2, and it's free to download.
- [HL7 v3 Product Brief](https://www.hl7.org/implement/standards/product_brief.cfm?product_id=186) -
  The ambitious RIM-based rewrite that largely failed to displace v2. Worth knowing about mostly so
  you understand why CDA (its surviving child) looks the way it does.
- [HL7 Confluence](https://confluence.hl7.org/) - Working-group specs, ballot material, and the
  v2-to-FHIR mapping tables. Less polished, more current.

## Integration Engines

In March 2025 NextGen took Mirth Connect closed-source as of version 4.6. **4.5.2 was the last open
release.** The community forked it twice, and both forks are alive and actively developed — this is
the single most important fact in HL7 tooling right now.

- [Open Integration Engine (OIE)](https://github.com/OpenIntegrationEngine/engine) - The
  community-governed fork of Mirth 4.5.2, with a steering committee and an application to join the
  Eclipse Foundation. 4.6.0 GA shipped July 2026 (Java 17, 24 CVEs remediated).
  [Site](https://openintegrationengine.org/) · [Docs](https://docs.openintegrationengine.org/) ·
  [Discord](https://discord.gg/azdehW2Zrx) (the primary community channel).
- [BridgeLink](https://github.com/Innovar-Healthcare/BridgeLink) - The other Mirth 4.5.2 fork, from
  Innovar Healthcare. Both forks are legitimate, both are actively developed, and there is little
  practical difference between them today — they share the same 4.5.2 ancestry and the same channel
  format. The distinction is governance (OIE has a steering committee and an Eclipse Foundation
  application; BridgeLink has single-vendor stewardship), which may matter more over time than it
  does right now.
- [Iguana](https://www.interfaceware.com/) - iNTERFACEWARE's engine, the main commercial
  non-Mirth comparator. (commercial)
- [NextGen Connect](https://github.com/nextgenhealthcare/connect) - The original. Frozen at 4.5.2
  since September 2024 and now closed-source going forward. **Do not start new work here** — use one
  of the forks. Kept in the list for archaeology and because a decade of Stack Overflow answers point
  at it.
- [OIE Vendors](https://github.com/OpenIntegrationEngine/vendors) - Who sells commercial support for
  OIE, if you need a throat to choke.

## Engine Plugins and Tooling

- [Mirth-Snippets](https://github.com/pacmano1/Mirth-Snippets) - Assorted OIE/Mirth scripts,
  templates, and how-tos. One of the most useful community repos.
- [Unofficial OIE Wiki](https://github.com/pacmano1/unofficial-oie-wiki) - Community-maintained user
  manual, filling the gap the official docs haven't yet.
- [ballista](https://github.com/kayyagari/ballista) - Admin-client launcher for Mirth/OIE/BridgeLink.
  Solves the Java Web Start pain that everyone hits on day one.
- [TLS Manager Plugin](https://github.com/NovaMap-Health/tls-manager-plugin) - Free TLS for HTTP/WS/TCP
  connectors, sponsored by NovaMap + Diridium and donated to OIE. The open replacement for NextGen's
  paid SSL Manager. [Docs](https://www.novamap.health/docs/tls-plugin)
- [Channel History Git Plugin](https://github.com/Innovar-Healthcare/Channel-History-Git-Plugin) -
  Git-backed version control for channels. Because "who changed this transformer" should have an
  answer.
- [ChristopherSchultz/mirth-plugins](https://github.com/ChristopherSchultz/mirth-plugins) - A solid
  grab-bag of community plugins.
- [mirth-plugin-guide](https://github.com/kpalang/mirth-plugin-guide) - The de-facto guide to writing
  a Mirth/OIE plugin, plus a [sample plugin](https://github.com/kpalang/mirth-sample-plugin) and a
  [Maven plugin](https://github.com/kpalang/mirth-plugin-maven-plugin) to build one. Dormant since
  2024 but still the best reference that exists.
- [Mike's Mirth Code](https://github.com/MichaelLeeHobbs/mmc) - Battle-tested JavaScript code
  templates, helpers, and channels pulled from production HL7 integrations. *(author of this list)*
- [oie-tcp-multi-sender](https://github.com/MichaelLeeHobbs/oie-tcp-multi-sender) - Destination
  connector that sends MLLP to a list of endpoints with failover/failback and a per-endpoint circuit
  breaker, instead of the stock TCP Sender's single address. *(author of this list)*
- [jonbartels' SSL/TLS primer](https://gist.github.com/jonbartels/8abd121901eb930f46245d9ef0f5710e) -
  The canonical explainer for Mirth keystore handling. Read before you fight the keystore.
- [HL7 Spec Extractor](https://github.com/Innovar-Healthcare/HL7_Spec_Extractor) - Extracts HL7 spec
  tables — useful if you're generating code or validators from the standard.

## Libraries

- **Java** — [HAPI HL7v2](https://github.com/hapifhir/hapi-hl7v2) - The reference implementation, and
  what Mirth/OIE uses internally. If you're on the JVM, this is the answer.
  [Docs](https://hapifhir.github.io/hapi-hl7v2/)
- **.NET** — [nHapi](https://github.com/nHapiNET/nHapi) - The .NET port of HAPI. Actively maintained;
  the clear pick.
- **Python** — [python-hl7](https://github.com/johnpaulett/python-hl7) - Simple parsing plus an MLLP
  client. Start here.
- **Python** — [hl7apy](https://github.com/crs4/hl7apy) - Structure-aware: validates against the
  actual v2.x grammars (2.1–2.8.2). Heavier than python-hl7, and worth it when you need real
  validation.
- **Node/TypeScript** — [node-hl7](https://github.com/Bugs5382/node-hl7) - Client, server, and parser
  in TypeScript. The actively maintained JS option — note the standalone `node-hl7-client` repo is
  archived; development continues in this monorepo.
- **Node/JavaScript** — [simple-hl7](https://github.com/hitgeek/simple-hl7) - Express-style HL7
  middleware plus MLLP. Low-maintenance but it works and it's small.
- **TypeScript types** —
  [mirth-connect-types](https://github.com/MichaelLeeHobbs/mirth-connect-types) - Type definitions for
  the Mirth/OIE server-side JavaScript (Rhino) User API — the globals and Java classes available
  inside channel scripts. *(author of this list)*
- **TypeScript** — [integration-engine-api](https://github.com/MichaelLeeHobbs/integration-engine-api) -
  Type-safe REST API client for NextGen Connect, OIE, and BridgeLink. *(author of this list)*

## Tools and Validators

- [HL7 Soup](https://www.hl7soup.com/) - Windows HL7 editor, viewer, and integration host. Free
  viewer tier. (partly commercial)
- [7Edit](https://7edit.com/) - Long-standing HL7 v2 editor/sender/receiver. (commercial)
- [Caristix View/Search/Edit](https://caristix.com/hl7-view-search-edit/) - HL7 workbench with
  conformance profiling. (commercial)
- [HL7 Inspector Neo](https://www.hl7inspector.com/) - Free browser-based HL7 analyzer and editor.
- [HL7 Viewer](https://www.hl7viewer.com/) - Free client-side parser. Nothing is uploaded.
- [HL7Toolbox](https://hl7toolbox.com) - Free online HL7 v2.x tools including an HL7 parser, message viewer, segment explorer, validation utilities, and developer resources for healthcare integration.

> **Do not paste production HL7 into a web tool you don't control.** Not the viewers above, not any
> of them. "100% client-side, nothing leaves your browser" is a *claim*, and it is routinely made by
> pages that also load third-party analytics and session-replay scripts (Microsoft Clarity, FullStory,
> Hotjar) — which exist precisely to record DOM content and ship it somewhere else. Unless the
> masking configuration is documented and the source is published, "client-side" and "nothing is
> captured" are not the same sentence.
>
> This bites hardest on the tools that sound safest: an **online PHI de-identifier is a contradiction
> in terms**, because to de-identify your data it must first receive your data, while it is still
> identified. Use a local tool — a library, a CLI, or something running on your own machine. If you
> must use a hosted tool, feed it [synthetic data](#test-data) and nothing else. A HIPAA breach is not
> worth the ninety seconds you saved.

## Test Data

- [Synthea](https://github.com/synthetichealth/synthea) - Synthetic patient generator producing full
  longitudinal records as HL7 v2, FHIR, and CCDA. The answer to "where do I get realistic test data
  that isn't PHI." [Site](https://synthetichealth.github.io/synthea/)

## FHIR

FHIR is not a replacement for v2 so much as the thing v2 shops eventually have to interoperate with.
A few canonical starting points, deliberately not a full FHIR list:

- [FHIR Specification](https://www.hl7.org/fhir/) - The spec itself.
- [HL7 v2-to-FHIR Implementation Guide](https://build.fhir.org/ig/HL7/v2-to-fhir/) - The official
  segment/field mapping guide. This is the bridge between the two halves of your career.
- [HAPI FHIR](https://hapifhir.io/) - The de-facto open-source Java FHIR client/server.
- [Firely .NET SDK](https://github.com/FirelyTeam/firely-net-sdk) - The official .NET FHIR SDK.
- [Inferno](https://inferno.healthit.gov/) - ONC's FHIR/US Core conformance test kit.
- [LinuxForHealth hl7v2-fhir-converter](https://github.com/LinuxForHealth/hl7v2-fhir-converter) and
  [Microsoft FHIR-Converter](https://github.com/microsoft/FHIR-Converter) - v2 → FHIR conversion, if
  you'd rather not hand-map every segment.

## Learning

- [Zen Healthcare IT — Mirth Connect Resource Center](https://consultzen.com/mirth-connect-resource-center/) -
  The best free Mirth tutorial hub. [YouTube](https://www.youtube.com/@ZenHealthcareIT)
- [Saravanan Subramanian — HL7 tutorials](https://saravanansubramanian.com/hl7/) - The best free
  written introduction to HL7 v2/v3 programming, with Java (HAPI) and .NET (nHapi) tracks.
  [Code](https://github.com/SaravananSubramanian/hl7)
- [Meditecs Mirth KB](https://www.meditecs.com/kb/mirth-connect-tutorial/) - Practical Mirth/OIE
  how-tos, including a clear explainer of the license change.
- [Mirth community forums](https://forums.mirthproject.io/) - A decade-deep Q&A goldmine. Still
  online, though the pulse has moved to the OIE Discord.
- **Book:** [Principles of Health Interoperability: FHIR, HL7 and SNOMED CT](https://link.springer.com/book/10.1007/978-3-030-56883-2)
  — Tim Benson & Grahame Grieve. The standard textbook, and Grieve *is* FHIR.

---

# DICOM

## DICOM Notes and Tips

DICOM punishes assumptions. These are the assumptions it punishes most often.

- **Transfer syntax is negotiated per presentation context, not per association.** The SCU proposes an
  abstract syntax with N transfer syntaxes; the SCP accepts exactly one per context. Always propose
  Implicit VR Little Endian as a fallback, or watch your contexts get rejected outright.
- **Whoever accepts a compressed transfer syntax owns the transcoding.** DCMTK needs `dcmjpeg`/`dcmjls`;
  pydicom needs `pylibjpeg`/`gdcm`/`pillow`. "It works with my sample file" almost always means the
  sample was Explicit VR LE and you have not yet met JPEG 2000.
- **C-STORE and STOW-RS fail differently.** C-STORE is a stateful association with per-instance status
  codes. STOW-RS is stateless HTTP with **no negotiation**, and it reports failures *inside the
  response body* (`FailedSOPSequence`) — often under a `200`. Parse the body; never trust the HTTP
  status alone.
- **DICOMweb has no C-MOVE.** Retrieval is a pull (WADO-RS). If a legacy modality can only C-MOVE, you
  need a proxy, not a URL.
- **AE Titles: 16 characters, no backslash, and vendors are fussy about case and whitespace.** A
  mismatched AE Title is the number-one cause of "association rejected."
- **Implicit VR plus private tags equals an unparseable file.** With no VR on the wire, an unknown
  private tag decodes as `UN`, and once it's `UN` your sequence and length assumptions are gone.
- **Never hardcode a private tag number.** Private elements are block-allocated: read the Private
  Creator element at `(gggg,0010–00FF)`, match its string, *then* compute the offset. Vendors move
  blocks between software versions, and your hardcoded tag will silently read someone else's data.
- **Accession Number `(0008,0050)` is `SH` — 16 characters, hard stop.** A RIS that mints longer
  accessions will get them truncated or rejected somewhere downstream, far from the source. (Same
  class of bug: Patient ID is `LO`, 64.)
- **`DS` and `IS` are *text* numbers.** Leading zeros, exponents, and whitespace are all legal. Don't
  assume your language's parser and the modality's writer agree.
- **`PN` is not a string.** It's five `^`-delimited components across up to three `=`-delimited group
  encodings (alphabetic/ideographic/phonetic). Treating it as a name will eventually mangle someone's
  name.
- **Character sets will find you.** `SpecificCharacterSet (0008,0005)` can be multi-valued with ISO
  2022 escape sequences. The sane modern answer is `ISO_IR 192` (UTF-8) — and since DICOM JSON is
  UTF-8 by definition, DICOMweb quietly forces the issue for you.
- **If you change pixel data, you mint a new SOP Instance UID.** Reusing one is the classic "PACS is
  still showing the old image and insists it's correct" bug. UIDs are ≤64 chars, digits and dots, no
  leading zeros per component; derive from your org root or use the `2.25.<uuid-as-integer>` root.
- **Pixel data lies in several directions at once:** `MONOCHROME1` is inverted; apply Rescale
  Slope/Intercept *before* window/level; `PixelRepresentation` decides signed vs unsigned; JPEG color
  is usually YBR, not RGB.
- **Do not sort slices by InstanceNumber.** Sort by `ImagePositionPatient` projected onto the normal of
  `ImageOrientationPatient`. InstanceNumber is a suggestion; geometry is a fact.
- **De-identification is not "strip the tags."** Burned-in pixel annotation
  (`BurnedInAnnotation (0028,0301)`), private tags, and SR content all leak PHI. Follow the PS3.15
  Annex E profiles, and keep a *consistent* UID mapping or your de-identified study won't hang
  together as a study anymore.

## The DICOM Standard

- [DICOM Standard (current)](https://www.dicomstandard.org/current) - The official current edition
  (2026c at time of writing). Parts 1–22; there is no Part 9 or 13 — they were retired.
- [PS3.5 — Data Structures and Encoding](https://dicom.nema.org/medical/dicom/current/output/html/part05.html) -
  VRs, transfer syntaxes, character sets, UID rules. The part you will actually read.
- [PS3.6 — Data Dictionary](https://dicom.nema.org/medical/dicom/current/output/html/part06.html) -
  Every standard tag, VR, and value multiplicity.
- [PS3.18 — Web Services](https://dicom.nema.org/medical/dicom/current/output/html/part18.html) - The
  normative DICOMweb spec (WADO-RS / QIDO-RS / STOW-RS / UPS-RS) and the DICOM JSON model.
- [Innolitics DICOM Standard Browser](https://dicom.innolitics.com/ciods) - By far the most pleasant
  way to navigate IODs, modules, and attributes. Note it's synced to the 2024 edition — great for
  learning, don't cite it as normative.
- [innolitics/dicom-standard](https://github.com/innolitics/dicom-standard) - The standard parsed into
  JSON. If you're building tooling, start from this instead of scraping NEMA.
- [dcm4che DICOMweb guide](https://dcm4che.github.io/dicomweb/) - The clearest DICOMweb walkthrough
  anywhere, with working curl examples. A far better starting point than PS3.18.

## Toolkits and Libraries

- **C++** — [DCMTK](https://github.com/DCMTK/dcmtk) - The reference toolkit and the canonical CLI
  tools. Effectively the coreutils of DICOM. [Site](https://dicom.offis.de/dcmtk)
- **C++** — [GDCM](https://github.com/malaterre/GDCM) - The strongest codec and transcoding coverage,
  with Python/Java/C# wrappers.
- **Java** — [dcm4che](https://github.com/dcm4che/dcm4che) - The Java toolkit beneath dcm4chee, plus a
  rich CLI suite of its own.
- **Java** — [PixelMed](https://www.pixelmed.com/) - David Clunie's toolkit. The site looks like 2003
  and the code is more authoritative than most things written since.
- **Python** — [pydicom](https://github.com/pydicom/pydicom) - The Python DICOM file library. Pair with
  [pynetdicom](https://github.com/pydicom/pynetdicom) for DIMSE networking (SCU/SCP) and CLI apps.
- **Python** — [highdicom](https://github.com/ImagingDataCommons/highdicom) - High-level construction of
  *standard-correct* derived objects (SEG, SR, annotations, parametric maps). Writing these by hand is
  how you produce files that only your own viewer can open.
- **.NET** — [fo-dicom](https://github.com/fo-dicom/fo-dicom) - Fellow Oak DICOM. Files, DIMSE, and
  rendering; the .NET pick.
- **JavaScript** — [Cornerstone3D](https://github.com/cornerstonejs/cornerstone3D) - The modern web
  medical-imaging rendering and tools framework (volumes, MPR, segmentation).
- **JavaScript** — [dcmjs](https://github.com/dcmjs-org/dcmjs) - DICOM read/write/derive in JS; the
  serialization layer under OHIF.
- **JavaScript** — [dicomParser](https://github.com/cornerstonejs/dicomParser) - Small, fast browser
  DICOM P10 parser. Still the go-to for "just read some tags." (maintenance mode)
- **Node.js** — [dcmjs-dimse](https://github.com/PantelisGeorgiadis/dcmjs-dimse) - C-ECHO/FIND/STORE/MOVE
  for Node. Fills the classic-networking gap in the JS ecosystem.
- **Node.js** — [dcmtk.js](https://github.com/MichaelLeeHobbs/dcmtk.js) - Type-safe Node bindings for
  the DCMTK CLI tools: wraps 51 binaries, long-lived server processes, a pooled auto-scaling
  `DicomReceiver`, and a queued `DicomSender` with backpressure. *(author of this list)*
- **Node.js** — [multipart-stream](https://github.com/MichaelLeeHobbs/multipart-stream) - Streaming
  consumer for `multipart/related` responses — i.e. what WADO-RS actually hands you — with proper
  timeout/abort/cleanup hygiene. *(author of this list)*
- **Rust** — [DICOM-rs](https://github.com/Enet4/dicom-rs) - Pure-Rust parsing and networking,
  including TLS.
- **Go** — [suyashkumar/dicom](https://github.com/suyashkumar/dicom) - High-performance Go parser plus a
  `dicomutil` CLI.

## Servers and Archives

- [Orthanc](https://orthanc.uclouvain.be/) - Lightweight single-binary DICOM server with a REST API and
  a strong plugin ecosystem (DICOMweb, PostgreSQL, OHIF, Python). The default answer to "I just need a
  PACS by lunch." [The Orthanc Book](https://orthanc.uclouvain.be/book/)
- [dcm4chee-arc-light](https://github.com/dcm4che/dcm4chee-arc-light) - Full enterprise PACS/VNA: HL7,
  IOCM, FHIR, UPS worklist. The heaviest and the most complete.
- [mercure](https://github.com/mercure-imaging/mercure) - A DICOM *orchestrator*: rule-based routing
  into containerized processing/AI modules. The interesting new entry in this category.
- [Microsoft dicom-server](https://github.com/microsoft/dicom-server) - Open-source .NET DICOMweb
  server; the engine behind Azure Health Data Services.
- [Dicoogle](https://github.com/bioinformatics-ua/dicoogle) - Indexing-focused open PACS with a plugin
  SDK. Strong for research and query-heavy workloads.
- [Conquest DICOM Server](https://github.com/marcelvanherk/Conquest-DICOM-Server) - Tiny, scriptable,
  Windows-friendly, still quietly running in radiotherapy departments worldwide. (legacy but alive)

## Viewers

- [OHIF Viewer](https://github.com/OHIF/Viewers) - The zero-footprint web viewer. DICOMweb-native,
  extension-based, and the de-facto standard for browser viewing. [Docs](https://docs.ohif.org/)
- [Weasis](https://github.com/nroduit/Weasis) - Desktop Java viewer, launchable from a portal. PET/CT
  fusion, curved MPR, genuinely capable. [Site](https://weasis.org/)
- [3D Slicer](https://www.slicer.org/) - Research imaging platform — segmentation, registration, a huge
  extension library, and a real DICOM database underneath.
- [VolView](https://github.com/Kitware/VolView) - Kitware's browser-based 3D volume renderer and
  annotator. Data stays client-side.
- [DWV](https://github.com/ivmartel/dwv) - Small, dependency-light JS viewer. The good embedding and
  teaching option.
- [Horos](https://horosproject.org/) - The macOS open-source viewer (an OsiriX fork; OsiriX itself went
  commercial). Now supports Apple Silicon.
- [MicroDicom](https://www.microdicom.com/) - Capable Windows viewer, free for non-commercial use.
  (freeware, not open source)

## DICOM Tools and Utilities

- [DCMTK command-line tools](https://support.dcmtk.org/docs/) - `storescu`, `storescp`, `findscu`,
  `movescu`, `dcmdump`, `dcmodify`, `dcmconv`, `img2dcm`, `dcmqrscp`. The test-and-simulate workhorses;
  learn these before you write any code.
- [pynetdicom apps](https://github.com/pydicom/pynetdicom/tree/main/pynetdicom/apps) - `echoscu`,
  `storescp`, `qrscp` and friends, pip-installable and runnable as `python -m pynetdicom echoscu`.
  Ideal for spinning up an SCP inside CI.
- [nodejs-dcmtk](https://github.com/MichaelLeeHobbs/nodejs-dcmtk) - Docker image: Node.js on Alpine with
  DCMTK compiled from source with full library support (XML, zlib, PNG, TIFF). *(author of this list)*
- [RSNA CTP](https://github.com/johnperry/CTP) - The Clinical Trial Processor: the industry-standard
  de-identification and routing pipeline for trials.
- [RSNA Anonymizer](https://rsna.github.io/anonymizer/1_overview.html) - The standalone "just
  de-identify this folder" utility.
- [pydicom/deid](https://github.com/pydicom/deid) - Recipe-driven Python de-identification, including
  pixel scrubbing.
- [KitwareMedical/dicom-anonymizer](https://github.com/KitwareMedical/dicom-anonymizer) - PS3.15
  profile-based Python anonymizer that's easy to embed.
- [DicomCleaner](https://www.dclunie.com/pixelmed/software/webstart/DicomCleanerUsage.html) - Clunie's
  GUI import/clean/save de-identifier, including pixel-data blackout.
- [dicom-validator](https://github.com/pydicom/dicom-validator) - `validate_iods` checks a file against
  the IOD/module requirements of a chosen standard edition. Pure Python and CI-friendly.
- [dciodvfy](https://www.dclunie.com/dicom3tools/dciodvfy.html) - The reference IOD validator. Grumpy,
  precise, and still the gold standard. It and `dicom-validator` disagree in places — if compliance
  matters, run both.
- [DVTk](https://github.com/dvtk-org/DVTk) - Protocol- and message-level validation plus emulators.
- [dicomweb-client](https://github.com/ImagingDataCommons/dicomweb-client) - Python DICOMweb client
  (QIDO/WADO/STOW). The easiest way to script against an archive.
- [dcm2niix](https://github.com/rordenlab/dcm2niix) - DICOM → NIfTI/BIDS conversion; the standard in
  neuroimaging.

## DICOM Learning

- [DICOM is Easy](https://dicomiseasy.blogspot.com/) - Roni Zaharia's long-running blog. The best
  "explain it like a programmer" introduction to DICOM, and still being written.
- [Innolitics DICOM articles](https://innolitics.com/articles/dicom/) - A genuinely good series on why
  DICOM exists, how DCMTK fits together, and what Part 15 means.
- [Saravanan Subramanian — DICOM tutorials](https://saravanansubramanian.com/dicomtutorials/) - Hands-on
  Java (dcm4che) and C# (fo-dicom) tracks: C-ECHO → associations → C-FIND/C-MOVE → C-STORE.
- [The Orthanc Book](https://orthanc.uclouvain.be/book/) - Its "Understanding DICOM" chapters are a
  practical primer well beyond Orthanc itself.
- **Book:** *Digital Imaging and Communications in Medicine (DICOM): A Practical Introduction and
  Survival Guide* — Oleg S. Pianykh, Springer. Still the standard printed introduction.
- [open-dicom/awesome-dicom](https://github.com/open-dicom/awesome-dicom) - A broader (if less curated)
  DICOM list. If something you need isn't here, look there.

---

# Security

Healthcare protocols were designed for a trusted network that has never existed.

- [DICOM PS3.15 — Security and System Management](https://dicom.nema.org/medical/dicom/current/output/html/part15.html) -
  The normative security part: TLS (BCP 195) profiles, the de-identification confidentiality profiles,
  digital signatures, and audit trail.
- **Both protocols are plaintext by default.** MLLP has no transport security at all. DICOM classic
  networking runs cleartext on 104/11112 (DICOM-TLS is 2762, and vendor support is uneven). The
  realistic mitigation is network isolation, VPN, and strict AE-title/IP allow-listing — plus TLS
  wherever both ends genuinely support it.
- [Millions of patient records at risk (Aplite, Black Hat EU 2023)](https://aplite.de/research/millions-of-patient-records-at-risk) -
  3,800+ internet-exposed DICOM servers, ~16M patients, and **under 1%** with proper authorization.
  [Press coverage](https://techcrunch.com/2023/12/06/medical-scans-health-records-dicom-pacs-security/)
- [CVE-2019-11687 — malware in the DICOM preamble](https://nvd.nist.gov/vuln/detail/CVE-2019-11687) -
  The 128-byte preamble happily accepts a PE header, so a file can be both a valid `.dcm` *and* a
  working Windows executable — malware living inside PHI that AV can't quarantine without HIPAA
  consequences. [Writeup](https://cylera.com/blog/weaponized-medical-images-is-malware-hiding-in-your-mri-results/)
- **DICOM parsers are attack surface.** Orthanc 1.12.11 (April 2026) alone fixed nine CVEs in parsing
  and decompression, and CISA issued a medical advisory for DCMTK in 2026. Treat untrusted DICOM the
  way you'd treat untrusted media files — because that's what it is.
- [Trend Micro — exposed DICOM servers](https://www.trendmicro.com/vinfo/us/security/news/cybercrime-and-digital-threats/a-hidden-vulnerability-in-healthcare-exposed-dicom-servers-and-the-risk-to-patient-data) -
  Background on how these ended up on Shodan in the first place.

---

# Dead ends

Things that rank well in search results and will waste your time. Each one has a live successor.

| Don't use | Why | Use instead |
|---|---|---|
| [hl7-standard](https://github.com/ironbridgecorp/hl7-standard) (npm) | 53★ and prominent in search; last release 2022 | [node-hl7](https://github.com/Bugs5382/node-hl7) or [simple-hl7](https://github.com/hitgeek/simple-hl7) |
| `Bugs5382/node-hl7-client` (standalone repo) | Archived; the npm package now ships from the monorepo | [node-hl7](https://github.com/Bugs5382/node-hl7) |
| [mirthstunnel](https://github.com/pacmano1/mirthstunnel) | Archived Feb 2026 | [TLS Manager Plugin](https://github.com/NovaMap-Health/tls-manager-plugin) |
| `tobchen/tc-ssl-plugin` | Abandoned since 2022 | [TLS Manager Plugin](https://github.com/NovaMap-Health/tls-manager-plugin) |
| `tonygermano/mirth-user-privacy-plugin` | Archived 2025 | — |
| `elomagic/hl7inspector` (Java desktop) | Archived 2020 | [HL7 Inspector Neo](https://www.hl7inspector.com/) (web) |
| [Imebra](https://dicomhero.com/) | Rebranded to DicomHero and went commercial | [DCMTK](https://github.com/DCMTK/dcmtk), [GDCM](https://github.com/malaterre/GDCM), or [fo-dicom](https://github.com/fo-dicom/fo-dicom) |
| [DICOMcloud](https://github.com/DICOMcloud/DICOMcloud) | Dormant since 2023 | [Microsoft dicom-server](https://github.com/microsoft/dicom-server) |
| Starting new work on [NextGen Connect](https://github.com/nextgenhealthcare/connect) | Closed-source as of 4.6; OSS line frozen at 4.5.2 | [OIE](https://github.com/OpenIntegrationEngine/engine) or [BridgeLink](https://github.com/Innovar-Healthcare/BridgeLink) |

---

## Contributing

Contributions are welcome — please read [CONTRIBUTING.md](CONTRIBUTING.md) first. The short version:
be honest about maintenance status, disclose it if you're submitting your own project, and understand
that tools touching PHI get extra scrutiny.

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/88x31.png)](https://creativecommons.org/publicdomain/zero/1.0/)

To the extent possible under law, [Michael Lee Hobbs](https://github.com/MichaelLeeHobbs) has waived
all copyright and related or neighboring rights to this work.
