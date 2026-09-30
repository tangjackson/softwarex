---
tags: [nsf, proposal, related-work, survey]
status: draft v1
created: 2026-09-30
solicitation: NSF 26-512 (AI Datasets)
---

> Organized by the two-layer research process from the whiteboard sketches (IMG_5062, IMG_5063): **I. Data infrastructure** and **II. Wireless science discovery** (II-A measurement conduction; II-B analysis, incl. the reverse-system loop), driven by a formal problem statement. Related: [[PD-CONTRIBUTIONS-DRAFT]] · [[NSF-PROBLEM-FORMULATION]]
> Citations were checked for existence and venue. Numbers about our own work come from the vault and hold on one build (XCAP-M 5.33.0.15) and pilot data. Appendices A-D at the end give the detailed fundamental problems, contributions, claims to avoid and the figure specification.

# Related Work: Metrology for Wireless Data

**Scope and method.** This survey follows the research process on the group's two whiteboards (IMG_5062, IMG_5063). It redraws them as one driver and two layers. The **driver** is the whiteboard's problem statement: diagnose failures, meaning (a) identify application failures (which requires outside or application data), (b) find the wireless reasons for them, and (c) relate those reasons to architecture decisions. **Layer I, data infrastructure**, covers hosting, metadata and provenance, readiness, verification and release. **Layer II, wireless science discovery**, has two parts. **II-A, measurement conduction**, covers conducting experiments and the chain that turns radio conditions into exported numbers. **II-B, analysis**, has two parts of its own. **II-B1, data analysis**, is extraction into tabular KPIs and signaling text messages, semantic extraction against the problem statement, and hypothesis validation. **II-B2, the reverse system**, is the group's loop for inferring the system from measurements: (a) measure application performance, (b) hypothesize about the system, (c) validate or invalidate with data, then return to (b). AI for scientific discovery runs across both layers.

Works were located through the Crossref, arXiv, Semantic Scholar, OpenAlex, NSF award and GitHub APIs, and each URL was fetched. Web search was exhausted in some sweeps, so every "we found no" below rests on that narrower search and should be rechecked before submission. Numbers about our own data come from the vault: the Project Description draft (nsf/PD-CONTRIBUTIONS-DRAFT.md) and the weekly notes. All of them come from one tool build (XCAP-M 5.33.0.15, which post-processes XCAL logs) and from pilot data. Items marked **[to confirm]** need the PI's confirmation.

---

## 0. The picture, redrawn from the whiteboards

A drive test records **raw chipset diagnostic bytes**. Separately, and outside the tool, it records **application data** (II-A). A **closed post-processor** turns the bytes into tabular KPIs and decoded signaling messages. It applies a procedure θ (sampling grid, per-parameter binning rule and statistic, records-per-event rule, export time zone and clock, tool build) that its exports do not carry. The group's analysis (II-B) extracts from these tables and messages what bears on a failure question, and then runs the hypothesis loop.

The whiteboard states one assumption at the entry to analysis: *"set to highest fidelity; assumption: all data available."* The central observation of this survey is that the literature leaves exactly this assumption unchecked. Decoding the bytes is well served by open tools [1, 2, 3]. The physical quantities are defined and their UE accuracy is specified [4, 95]. Network-side counters declare their collection method per measurement [93]. Containers for provenance exist [6, 7, 22], and so do analysis tools [8] and agents [9]. What we found missing is the **procedure that gives each drive-test value its meaning**, recorded per field and checked. Metrology calls the resulting spread *definitional uncertainty* [10].

The loop stays the group's. The new element is an **instrument check at step (c)**: before a field is used to validate or invalidate a hypothesis, check that it carries a passport (C1), and that the procedure in it fits the hypothesis's time scale. If the procedure is unknown or unfit, the field goes to passive inference (C3) or, conditionally, to experiments on the closed tool (C5). What either recovers enters the passport only through the Two-Path Rule (C2). The passports make existing corpora comparable (C4), and the reverse loop then runs on values that carry their procedure. Network inference remains the loop's scientific target.

```
  PROBLEM STATEMENT (driver): diagnose failures
    (a) identify app failures [needs outside/app data]  (b) find the wireless reasons
    (c) relate them to architecture decisions (NSA/SA, band, TDD, CA)
                                   |
 II WIRELESS SCIENCE DISCOVERY     v
 +- II-A Measurement conduction ------+   +- II-B Analysis --------------------------------+
 | networks -> collectors -> RAW BYTES|   | II-B1 Data analysis                            |
 |  (+ app data, outside the tool)    |   |  extraction -> tabular KPIs + signaling msgs   |
 |        |                           |   |  -> semantic extraction wrt problem statement  |
 |  [closed post-processor: theta] ---+-->|  -> validate hypotheses?                       |
 |   grid, rule, statistic, event     |   |  entry: "highest fidelity, all data available" |
 |   rule, time zone/clock, build     |   | II-B2 Reverse system (the group's loop)        |
 |        |                           |   |  (a) measure app perf -> (b) hypothesis about  |
 |  RAW BYTES -> our container reader |   |  system -> (c) validate/invalidate -> (b) ...  |
 |   + open decoders  (C2 path)       |   |  NEW at (c): instrument check on each field    |
 +------------------|-----------------+   |  unknown/unfit -> C3 passive | C5 experiments  |
                    |                     +----------------------|-------------------------+
 I DATA INFRASTRUCTURE                                           v
   prior: archives, DOIs, Croissant 1.1 PROV-O, SOSA, RFC 8911/8193, TS 32.404, TS 38.215/38.133
   ours:  C1 Value Passport <-- only via C2 tiers --  C3 WiReady  -->  C4 comparable existing corpora
```

**Legend for the figure (status is encoded, not only stated in the caption):** grey = prior work; outlined = ours, published (the five campaigns and the network-inference studies); solid = ours, pilot evidence exists (the XCAL container reader, the -20.0 / -12.9 dB case, 37 vs 2,151, 743 of 1,171); dashed = ours, proposed (C0-C4); dotted = ours, conditional (C5).

**Where each whiteboard item went.** The PI can check that nothing was lost.

| Whiteboard item (IMG_5062 / IMG_5063) | Where it sits now | What addresses it |
|---|---|---|
| Problem Statement (1) diagnosing failures: (a) identify app failures, requires outside/app data; (b) find wireless reasons for failures; (c) relate to architecture decisions | Driver above II-B (Sec. 1) | FP0; C0 formalizes it; C4 part 5 (labels from application records) |
| (1) Conducting Exps: discover the minimal set of params required | II-A; for existing data, read as the minimal *fit* set of fields and settings for a question | C3 question-to-field fitness; C5-A2 dependency screening |
| (1) Conducting Exps: design experiments for system inference and diagnosing failures | II-A network side = the 2020-24 campaigns (not re-run: no new collection); tool side = experiments on the closed post-processor | C5 (conditional) |
| (2) Analysis: Extraction -> tabular / text msgs -> Validate Hypotheses? | II-B1 | C1 attaches a procedure to each tabular field; C4 part 4 event layer for messages |
| Semantic Extraction wrt Problem Statement | II-B1.1 | C3 question-to-field fitness (proposed) |
| Set to highest fidelity; assumption: all data available | Entry to II-B; tested per field | C1 records the setting; C3 flags gaps (e.g., packet capture off); FP1 rebuttal |
| (2) Inferring System: design from measurements; (a) measure app performance, (b) make hypotheses about the system, (c) validate/invalidate with data, loop back | II-B2, at the center, as the group's published method | Instrument check at (c), using C1 and C2 |
| Contribution (1) formalize problem statement definition | Sec. 4, FP1's staged model | C0 |
| Contribution (2) automated analysis? | II-B1.5 | C3 disclosure test; C4 re-tested findings; C5 (conditional). Automated cross-layer RCA itself is prior [8, 119] |
| Contribution (3) finding semantically relevant data | II-B1.1 | C3 fitness check, C5-A2 |
| Contribution (4) experiment procedure determination | II-B2.5 | Two readings: recovering the procedure behind existing data (C3, C5, into C1) is in scope; designing new network experiments is out of scope |

---

## 1. The driver: diagnosing application failures from wireless data

Diagnosing application failures from cross-layer data is established science. QoE Doctor diagnoses mobile-app QoE by driving app interfaces automatically and analyzing data from the app down to the radio (RRC) layer [121]. A nationwide study of cellular failures on Android phones measures and analyzes them, and proposes enhancements [122]. From cross-layer measurements, Yi et al. trace video-conferencing degradations to 24 previously unknown causal event chains and build Domino to automate that tracing [8]. This group relates throughput, latency, power and QoE to architecture decisions: NSA vs SA, band, TDD frame, carrier aggregation and MIMO rank [45, 19, 46, 20]. Persistent 5G ON-OFF loops have been traced to inconsistent network- and device-side triggers [48]. Crowdsourced speed tests have been shown to need context before they can be read as failures [42].

These works either capture application and radio data together, with synchronized clocks [121, 8], or treat derived radio fields as observations. The whiteboard's step "identify app failures" needs outside data, and so does every join to the radio side. In our pilot, radio and application data align only to within ±1.5 s for streaming and only to seconds for teleoperation. That is comparable to the 2.0 s median stall, so stall labels hold only at second level. And 1-second averages hide 10-ms SINR dips.

**Solved.** Cross-layer failure diagnosis when application and radio telemetry are captured together and clocks are synchronized; attributing performance to operator configuration.
**Open.** Attribution on legacy observational corpora, where the procedure behind each validating field is unrecorded. A test of whether a field's procedure can resolve the failure's time scale: we found none.

---

## Part I. Data infrastructure

### I.1 Archives, testbeds, platforms and public releases

Hosting, identifiers and citation for wireless data are mature. CRAWDAD has archived wireless traces since 2004 and moved to IEEE DataPort in 2022 [11]. PAWR testbeds make experiments with open stacks repeatable at city scale [12]. MONROE ran an open experimentation platform on commercial mobile broadband networks, with nodes that collect metadata such as modem and location information alongside experiment results [123]. DeepSense 6G co-captures mmWave power, camera, LiDAR, radar and GPS [13]. The Vienna 4G/5G Drive-Test Dataset (Scientific Data, 2026) pairs scanner and handset logs recorded with Keysight Nemo at an effective granularity of about 500 ms, and states that its timestamps are UTC [14]. The AERPAW curation harmonizes campaign-level metadata, including altitude, carrier frequency, bandwidth, sampling configuration and calibration status, and provides post-processing scripts [15]. Commercial-network corpora appear as paper artifacts, including this group's Lumos5G [16], 5Gophers [17], 5G mobility [18], 5G mid-band [19] and the joint Chicago/Minneapolis campaigns [20].

In these releases the procedure is described **once per dataset, in prose**. The Chicago/Minneapolis README says the data came from "XCAL, sampled over 1s by averaging numeric data and finding the mode of discrete data". It gives no XCAP version, no other settings and no export time zone [20]. Vienna states that "the raw proprietary Nemo Outdoor export files and campaign-specific export configurations are not included" [14]. Vienna does record its time base (UTC); our own releases do not. AERPAW compares Nemo with an Android app and reports that "measured throughput statistics depend on the logging tool". It does not reconcile the two tools' outputs or record per-field aggregation [15]. None of the five public releases whose top-level documentation we checked (three from our own campaigns) records its full export configuration.

**Solved.** Long-term hosting, DOIs and citation; large corpora at every scale; programmable testbeds and platforms that record node metadata; campaign-level harmonization, including the sampling configuration.
**Open.** The derivation procedure behind each field of values produced by closed post-processors: we found no release that records it. Cross-tool reconciliation: tool dependence is documented [15], but we found no archive that flags values as non-comparable. Raw, re-binnable logs are rarely released.

### I.2 Metadata, provenance and measurement-definition standards

Four families supply **containers, slots and declared rules**, each at a different level.

- **ML dataset metadata.** Croissant 1.1 (January 2026) attaches PROV-O provenance at dataset, resource, RecordSet, field and individual-value level [6]. Datasheets [21] structure documentation written by the producer. The FAIR principles [108] set findability and reuse goals.
- **Provenance, observation and quality vocabularies.** PROV-O models the procedure an agent followed as a `prov:Plan` [7]. SOSA/SSN defines `sosa:Procedure` ("a workflow, protocol, plan, algorithm, or computational method specifying how to make an Observation") [22]. The W3C Data Quality Vocabulary annotates datasets with quality measurements and certificates [101]. SigMF describes recorded RF sample files [102].
- **Network-measurement metric and result records.** The IETF registry records each metric's Method of Measurement, Output and version [5]. The LMAP information model attaches to every reported result the ordered options of the Task and Action that produced it, together with start and end times and any conflicting actions [98]. ETSI TS 102 250-6 fixes post-processing statistics and aggregation hierarchies. Its scope is service-level QoS measured by probing systems, especially in GSM and 3G, and it does not address UE radio parameters such as RSRP or SINR [23].
- **3GPP.** TS 32.404 gives every network performance-management counter a template: (a) Description; (b) Collection Method, one of CC, GAUGE, DER ("every nth event is registered"), SI, TF and OM; (c) Condition, the protocol trigger events that update the result; and (d) Measurement Result, with values and units [93]. That is a declared statistic and event rule per parameter, for **network-side** counters. Minimization of Drive Tests (TS 37.320) makes UE logging a configured measurement: for periodic logging "the logging interval is configurable" and "specifies the periodicity for storing MDT measurement results". Event triggers have configurable threshold, hysteresis and time-to-trigger, and "measurements in measurement logs shall be linked to a time stamp" [94]. MDT is collection configured by the operator through the network, not the academic drive-test path. TS 38.215 defines SS-RSRP and SS-SINR and leaves, for example, the number of resource elements used "up to the UE implementation" [4]. TS 38.133 sets accuracy requirements and measurement periods [95], and TS 38.533 tests them with known applied conditions [96].

So the network-counter layer and the UE-measurement layer are both specified. The gap is narrower: **how drive-test post-processors turn UE diagnostic logs into exported rows**. For that step we found no counterpart to the TS 32.404 template or to MDT's configured interval. Every scheme above also assumes the publisher knows the procedure. For a closed post-processor the producer usually does not: in our tool, declared rules cover 743 of 1,171 parameters, and 428 are blank. For this reason C1 is designed as a **measured instance of the TS 32.404 template fields**, mapped into Croissant 1.1 and PROV-O, and not as a new schema.

**Solved.** Containers with value-level provenance hooks; semantics for "the procedure used to make an observation"; per-result option records where the platform owns the pipeline [98]; declared per-counter collection methods and conditions [93]; configured UE logging in MDT [94]; physical definitions and UE accuracy requirements [4, 95, 96].
**Open.** Measured procedure content for drive-test post-processing of UE diagnostic logs, per parameter and per build (grid, statistic, records per event, time base, blind spots, sensitivity range, verification tier): we found none. An honest encoding of "implementation-defined" and "unknown" that AI consumers can condition on or refuse.

### I.3 AI readiness, verification and AI-ready platforms

NSF 26-512 asks for "automated platforms that discern the readiness of legacy datasets at scale" [24]. Lawrence's Data Readiness Levels name the question we target in Band B: "is what is recorded matching what is purported to be recorded?" [25]. The automated tools work at dataset level. AIDRIN scores completeness, outliers, imbalance, privacy and FAIR compliance over the table as given [26]. F-UJI automates FAIR assessment [109]. SetGo scores and repairs metadata on six dimensions, provenance among them [27].

Two funded CICI/IPAAI projects are the nearest analogs. CANIS verifies the fidelity of a network telescope with beacon packets and known scanning campaigns; it does this for an instrument it operates itself [28]. CERTIFY lets scientific data objects "carry machine-checkable proofs that specified integrity and provenance requirements are satisfied", compiles temporal contracts into OS-level monitors, and uses LLM agents to configure them [99]. That is closer to C2 than CANIS is. It certifies that *declared* transformations respect *declared* properties; it does not recover an undeclared procedure inside a closed tool. SLSA Verification Summary Attestations record who verified an artifact, against which policy, and with what result [100], and DQV can carry quality certificates [101]. Verification tiers are therefore an existing concept. What remains open is the certifying path, discussed in X.4.

AI-ready telecom datasets optimize for ML tasks rather than for what their values mean. TeleLogs is synthetic [29], and TelecomTS keeps absolute KPI scale but does not state per-KPI aggregation [30]. In our pilot, packet capture was off in 8 traces (3.3 GB), and this is stated nowhere. Dataset-level checkers do not test what a field means, so they would not catch such a gap. There is also a denominator problem. Our five-release documentation check (three ours) is not a survey of the field, so a Year-1 systematic audit of 15-20 public drive-test releases (IEEE DataPort/CRAWDAD, Zenodo, AERPAW, Vienna and others) for tool, build, settings, time zone and raw-log availability is proposed under C3.

**Solved.** Dataset-level scoring of quality, FAIR compliance and metadata completeness; certification of declared integrity and provenance properties [99, 100]; fidelity checks on self-operated instruments [28].
**Open.** Automated field-level Band-B readiness for legacy exports whose producer cannot answer: which procedures are consistent with this export, at what coverage, or "not inferable". We found no such tool. Readiness claims calibrated against ground truth, such as re-exports at known settings. A real denominator for how many public wireless releases came from closed post-processors.

### I.4 Reproducibility, versioning and tool dependence

Measurement research named the problem early. Paxson stressed associating meta-data with measurements, and calibrating them by examining outliers and testing for consistencies, including measuring a phenomenon more than one way [31]. A growing line of work documents tool dependence directly. Ookla and NDT7 diverge systematically across 80,000+ paired tests [32], and AERPAW finds throughput statistics depend on the logging tool [15]. In a preprint, Kaplan et al. compare web-GUI, AT-command and Accuver XCAL extraction on two commercial mmWave 5G CPEs. Under 0.25 s polling, AT commands respond every 0.25 s and XCAL every 1 s. RSRP is integer on the AT and GUI paths and floating point in XCAL. They rank XCAL best on value resolution and refresh rate, but do not record the procedure behind each value [33]. In an adjacent domain, a 2026 study finds that for industrial control system datasets "system topology, clock synchronization, labeling semantics, and preprocessing pipelines are all frequently underreported, even when artifacts are nominally shared" [34]. Versioning tracks changes to data (for example, Zenodo versions of the Vienna set [14]) but not changes in post-processor behavior. On our install, the version label reads 5.33.0.15 while the build paths read 5.33.0.9.

**Solved.** Artifact-release norms; principles of sound measurement; documented tool dependence; version mechanics (DOIs, checksums, semantic versioning in Croissant 1.1 [6]).
**Open.** Reproducibility checks on meaning, not just availability: a result can be "reproduced" under the same hidden default. Turning known tool dependence into per-value metadata. Versioning tool behavior per build.

### I.5 Synthetic and digital-twin data (contrast class)

Ray-tracing corpora are "AI-ready" because the generating procedure *is* the dataset: DeepMIMO is reproducible from its stated scenarios and parameters [35]. The weak point is real-world grounding. Differentiable ray tracers are calibrated against channel-sounder measurements whose processing is known [36]. A twin calibrated against commercial drive-test KPIs would inherit whatever undisclosed binning produced them.

**Solved.** Fully specified synthetic channel and link data; calibration against research-grade instruments.
**Open.** Procedure-matched comparison of simulated and exported KPIs (should a twin's per-slot SINR be compared with a 1 s minimum or a 1 s mean?), and metadata that marks a value as synthetic, measured with a known procedure, or measured with an unknown one.

---

## Part II. Wireless science discovery

### II-A Measurement conduction

#### II-A.1 The chain behind one number

A drive-test KPI passes through four stages before anyone analyzes it:

`v = A_post(b; θ)`, with `b = C_chip( F_L3( M_L1(x; ι); k ); λ )`

1. **M_L1(x; ι), UE L1 measurement.** TS 38.215 defines the quantity and leaves some choices ι "up to the UE implementation" [4]. TS 38.133 bounds its accuracy and measurement period [95], and TS 38.533 tests it on conducted setups [96]. *Class:* the definition is fixed by standard; ι is not identifiable from any record and can only be bounded.
2. **F_L3(·; k), RRC Layer-3 filter** for quantities used in measurement reporting and event evaluation: F_n = (1 − a)·F_(n−1) + a·M_n, with a = 1/2^(k/4). The coefficient k is the filterCoefficient the network signals in RRC configuration (TS 38.331, Sec. 5.5.3.2) [97]. *Class:* recoverable from the log, when those RRC messages are logged.
3. **C_chip(·; λ), chipset diagnostic logging.** This stage fixes which log codes are written, at what rate, and whether a code carries per-sample or filtered values. *Class:* partly recoverable from the log. We found no public documentation of which stage each code reports, so this belongs in the passport.
4. **A_post(·; θ), the post-processor.** θ is the sampling grid, the per-parameter binning rule and statistic, the records-versus-events rule, the export time zone and clock, and the build. *Class:* post-processor-specific, and recorded in no export we examined.

The formalization matters because it shows that **the finest export is not raw**. XCAP's finest setting ('All') still sits downstream of the UE's L1 measurement and, for reported quantities, of the network-configured L3 filter. The tool's own manual also states that the sampling interval "is not applied for Call Events and Signaling Messages". So a single export mixes sampled metrics with unsampled events and messages, which have two different time semantics.

**Solved.** Stage 1 (definitions, accuracy, conformance tests) and the mechanics of stage 2.
**Open.** Stages 3 and 4 as recorded metadata: which stage each exported field reflects, and under which θ.

#### II-A.2 Commercial collector plus closed post-processor

Many academic 5G datasets with PHY/MAC depth are made with a vendor toolchain, including this group's XCAL-based campaigns [19, 20] (tool split per campaign **[to confirm]**). A collector records chipset diagnostic logs from engineering phones, and a closed post-processor turns them into KPIs, maps and, increasingly, "AI-driven" root-cause analysis. Accuver's XCAP advertises "automated post-processing workflows" [37], and TEMS Discovery re-derives KPIs from other vendors' logs under its own procedure [38]. The vendor pages we read describe features, not how each exported parameter is computed. The settings do exist inside the tool's state: our AI assistant worked mostly by reading the tool's model files. But **exports do not carry the configuration that produced them**. Network counters (TS 32.404 [93]) and MDT logs (TS 37.320 [94]) declare their collection rules. For drive-test post-processing of UE diagnostic logs we found no standard or vendor document that does. Our evidence on one build:

- Worst SINR on one trace is -20.0 dB at the finest setting and -12.9 dB at the documented 1-second default, a fivefold gap in linear terms. This is aggregation arithmetic (1-second averages hide 10-ms dips), not a vendor defect.
- Counting table rows instead of events turns 37 handover attempts into 2,151. Message pairing pairs 62 of 62 handovers.
- Four documentation-behavior gaps bear on values (one was seen in a single bin).
- "XCAL data" spans four collection product lines, all analyzed by one XCAP build.

**Solved.** Collecting and decoding multi-chipset logs into thousands of KPIs (commercial capability); service-level statistics [23]; evidence that the extraction path changes latency and resolution [33].
**Open.** Per-parameter procedure content for closed drive-test post-processors: we found no vendor document or published study that records it. Version drift, with no public per-build change catalog. Built-in AI root-cause analysis layered on undisclosed KPI procedures.

#### II-A.3 Open decoders and OS APIs

Open tools decode the diagnostic stream without vendor software. MobileInsight extracts below-IP messages on phones and adds analyzers [1]. SCAT parses Qualcomm and Samsung diagnostics; its paper used about 3.1M control-plane messages from 13 operators to diagnose operator misconfigurations [2]. QCSuper captures raw frames over the Qualcomm Diag protocol [3]. This is why "decoding is largely solved" holds. What these tools solve is decoding messages and payloads, not how per-message values become a dataset column. Their analyzers are themselves procedures: NG-Scope reports a MobileInsight-based capacity estimator with 30.5 Mbit/s RMSE, against 9.2 Mbit/s for NG-Scope [39]. The Android API adds its own layer: NR SS-SINR is reported as an integer in -23 to 40 dB, and the timing-advance getter returns a 3GPP index, "not physical microseconds, despite the method name" [40].

Our own reader of XCAL's container counts the same records as the tool for 165 of 167 log codes on one ping trace. It decodes payloads for only 3 of at least 141 codes. It parsed 15.3 GB at 283 MB/s with zero resyncs, and all 15 of those traces are ping traces. A GitHub API search returned no public reader for the XCAL container, only GUI-automation utilities; the search was narrow. Two things are still open. One is whether .drm payloads are standard DIAG packets that SCAT, MobileInsight or QCSuper can parse. The other is whether our reader's decoding of the codes it matches was worked out from XCAP's own tables, which would fail the independence test by construction.

**Solved.** Open decoding of Qualcomm and Samsung diagnostics; non-root access to OS radio KPIs with stated ranges.
**Open.** Aggregation semantics from decoded message to column; decoder-version dependence; partial open coverage of NR L1/L2 payloads; a public reader for vendor containers such as XCAL's.

#### II-A.4 Independent observers and known-answer references

Over-the-air telemetry observes the base station's control channel with an SDR, independent of the UE chipset and of vendor software. NG-Scope estimates LTE capacity under carrier aggregation [39]. NR-Scope decodes 5G SA DCI, SIB and RACH to estimate per-UE PRB allocation and bit rate, with under 0.1% per-UE throughput error on the gNBs it was tested on [41]. These are independent references for **scheduler-visible** quantities: PRB, MCS/TBS, retransmissions and per-RNTI throughput. Testbeds such as POWDER expose gNB-side ground truth [12].

For **UE-internal** fields, independent references exist for the *physical quantity*. Scanners are independent receivers measuring the same SSBs (Vienna pairs scanner and handset logs [14]), and RRM conformance tests apply known SS-RSRP and SS-SINR to a UE through a base-station emulator [96]. What does not exist is a reference for *the specific value a field UE computed*: for that value the chipset log is the only primary record. We found no published study that uses a testbed, sniffer or emulator to calibrate a commercial post-processor's exported KPIs end to end.

**Solved.** Real-time decoding of LTE and 5G SA control channels; independent references for scheduler fields and for the physical quantity; conducted known-answer tests of UE accuracy.
**Open.** End-to-end calibration of the logging-plus-post-processing chain: known conditions applied, logged by the vendor collector, exported by the vendor post-processor. Open-sniffer coverage of NSA and mmWave.

#### II-A.5 Crowdsourced measurement and methodology bias

Crowdsourced tests bring scale together with two biases: who and where gets measured, and what the test procedure actually measures. Paul et al. show that many "slow" tests reflect subscription tier, and that Ookla medians can be up to 2x M-Lab's for the same tier, city and ISP because the methodologies differ [42]. MacMillan et al. quantify the divergence between the two methodologies [32]. That community has accepted, at dataset level, that the procedure must travel with the value. For radio KPIs the step has not been taken. Per our corpus inventory **[to confirm]**, our five corpora mix pipelines: two XCAL-based, one chipset-level and two phone-side app logs [16, 17, 18, 19, 20].

**Solved.** Recognizing and quantifying methodology-induced differences between speed-test tools.
**Open.** Per-value instrument and procedure labels for pooled radio KPIs: we found no crowdsourced mobile corpus that records the averaging window or statistic per field.

#### II-A.6 Calibration and the time base

Clock offset and skew can be estimated and removed from timestamps when both clocks are observable [43]. After-the-fact synchronization of streams also exists in other sensing domains: SyncWISE aligns video and accelerometry by window-induced shift estimation [104]. RF instruments are verified against a reference (a VNA) in a controlled channel, with propagated uncertainty [44]. So metrology exists **below** the gap (RF instruments, UE accuracy [95]) and **above** it (IP-metric registries [5], service-KPI post-processing [23], network counters [93]). At the step from UE diagnostic log to post-processed KPI we found no calibration or procedure-disclosure practice of comparable rigor. In our data, export times sit 5 hours behind chipset time in two collections and match it in a third, and this is recorded nowhere. The most plausible cause is an unrecorded export time zone, so this is an instance of θ rather than a deep clock problem. Phone-clock jitter measured 5,028.6 ms.

**Solved.** Clock calibration when both clocks are observable; after-the-fact synchronization of sensor streams [104]; comparison-to-reference verification of RF instruments.
**Open.** Radio-to-application alignment in legacy drive-test corpora, with a stated bound per join; the time base as a recorded field.

### II-B Analysis

#### II-B1 Data analysis

##### II-B1.1 Extraction, and semantic extraction with respect to the problem statement

The whiteboard's analysis starts with **extraction** into two forms: **tabular** KPIs (what XCAP exports) and **signaling text messages** (RRC/NAS, rendered by the vendor tool or by open decoders [1, 2]). Then comes **semantic extraction with respect to the problem statement**: choosing, from 1,171 exported parameters and at least 141 log codes, the fields that bear on a given failure question. Language models already help with the textual side of this step. Telco-RAG retrieves from 3GPP documents for telecom question answering [124]. TSpec-LLM releases the 3GPP specifications as an open dataset for LLMs [125]. SPEC5G is a dataset for 5G protocol analysis from specification text [126], and CellularLint finds inconsistent behavior in cellular specifications [127]. ConSeT decodes RRC messages into structured fields and infers cross-field dependencies with an LLM in an evidence-bounded way [128]. Doduo annotates table columns with types and relations using pre-trained language models [129].

These methods retrieve fields and spec text by *meaning*. A failure question also needs fields selected by *fitness*: whether the procedure behind a field can resolve the event (a 1-second mean cannot validate a 10-ms dip or date a 2-second stall to the second). We found no system that ranks drive-test fields by whether their procedure fits a question's time scale. With passports (C1), a field becomes retrievable by its procedure as well as by its name. The proposed **question-to-field fitness check** is part of C3, and C5-A2's dependency screening supplies which settings a field depends on.

**Solved.** Extraction of tabular KPIs and decoded messages; retrieval over specifications; LLM inference of field constraints in RRC messages; semantic column annotation.
**Open.** Selecting fields fit for a question by their measurement procedure.

##### II-B1.2 Cross-layer characterization and root-cause analysis

Top-venue studies explain 5G throughput, latency, power and QoE in terms of configuration [45, 19, 20]. Root-cause analysis has moved from manual studies to tools. From cross-layer measurements on private and commercial 5G cells, Yi et al. identify 24 previously unknown causal event chains that degrade video conferencing, and build Domino to automate the tracing. Their high-rate experiments use NR-Scope, add gNB logs on private cells, and synchronize all host clocks with NTP [8]. Every work here treats the derived KPI as the observation, and we found none that treats the post-processor's procedure as a factor. Legacy exports make the assumptions behind such pipelines fragile (Sec. 1).

**Solved.** Cross-layer alignment and RCA when telemetry is open and clocks are synchronized; attributing performance to operator configuration.
**Open.** RCA on legacy exports whose time base is uncertain; how sensitive published attributions are to the processing procedure.

##### II-B1.3 Prediction and mobility

Throughput prediction runs from Lumos5G's composable ML framework for mmWave [16] to Prism5G's CA-aware predictor [46]. MMLab crawled handoff configurations across 30 operators and 32,000+ cells [47]. This group's cross-country study (6,200+ km, three US carriers) dissected 5G handover mechanisms and built the Prognos predictor [18]. A 2025 study traces persistent 5G ON-OFF loops to inconsistent network- and device-side triggers [48]. These works count signaling events and leave the counting rule implicit. In our data, message pairing pairs 62 of 62 handovers, while counting table rows gives 2,151 "attempts" instead of 37. We found no prediction or mobility study that measures how accuracy, feature importance or event counts change when the same raw logs are post-processed under another documented procedure.

**Solved.** Accurate within-dataset prediction; recovering operator mobility configuration from device-side signaling.
**Open.** Machine-checkable event definitions released with counts; transfer across post-processors; flags on fields unfit for sub-second questions.

##### II-B1.4 Causal inference and anomaly detection

Causal methods now separate configuration effects from environmental confounders on observational telemetry from about 1M commercial cells [49]. They model confounding from algorithms and environment. None that we found models confounding from the **measurement function**. Without procedure metadata, a change of tool build or export setting between collections looks exactly like a network change.

**Solved.** Operator-scale anomaly detection; controlling for environmental and collection-policy confounders.
**Open.** The processing procedure as a node, confounder or intervention.

##### II-B1.5 ML, LLMs and agents on network data (the whiteboard's "automated analysis?")

One LLM can be adapted to several networking tasks [50]. Telecom benchmarks have moved from knowledge QA to data-grounded tasks: TeleLogs poses RCA on synthetic drive-test-like data with 8 root causes [29], and TelecomTS offers 10 Hz testbed KPIs with mostly injected anomalies [30]. Process-level audits of RCA agents on microservice software systems show the risk: across 11 frontier LLMs, OpenRCA 2.0 finds exact root-cause sets in 20.7% of cases, and names a correct service in 76.0% but grounds it in a verified causal path in only 61.5% [51].

Agentic measurement is arriving in the neighboring Internet-measurement community. ArachNet composes Internet measurement workflows with LLM agents [118]. Airavat adds "a Verification Engine [that] evaluates workflows against a knowledge graph encoding five decades of measurement research" and a Validation Engine that selects validation techniques [119], so the verifier is separate from the generating agents. Beltiukov et al. use LLMs and historical data to explain network measurements to non-experts [120]. Airavat checks a *workflow's methodology* against codified knowledge. C2 admits a *value* only when an independent data path that shares nothing but the raw bytes with the tool and the agent reproduces it. We found no network or telecom AI dataset that attaches per-value procedure metadata, or that asks a model to state the measurement condition behind a numeric answer. In our pilot, an AI assistant working from a shell solved 5 of 11 tool tasks fully and 5 partly, yet none of its 5 numeric answers named its setting (one model family, one grader, single seed).

**Solved.** Adapting LLMs to networking tasks; RCA benchmarks on synthetic or testbed data; agentic measurement workflows with methodology verification [119].
**Open.** Benchmarks on real UE drive-test exports with independently verified values; tests of condition disclosure; value-level admission.

##### II-B1.6 Threats to validity

ML-based science often over-claims through leakage; one survey found it in 17 fields, affecting 294 papers [52]. Label errors in test sets destabilize benchmarks [110]. Multiverse analysis shows that defensible analysis choices change conclusions [53]. Across 35 repeated drives on one Starlink route, single cherry-picked drives yield "significant" correlations that the full dataset shows are variation from uncontrollable factors [54]. These works attribute divergence to the environment or to modeling. Post-processing is the one component that can be **re-derived after the fact** from held raw logs, and we found no validity study that isolates it.

**Solved.** Leakage and label-error taxonomies; multiverse analysis for analyst choices; replicability under environmental variability.
**Open.** Sensitivity ranges for published radio-KPI findings across processing procedures; re-testing findings with processing held constant.

#### II-B2 The reverse system: inferring the system from measurements

##### II-B2.1 The group's loop, and inferring closed networks

The whiteboard's loop, (a) measure application performance, (b) hypothesize about the system, (c) validate or invalidate with data, then back to (b), is the group's published method [18, 46, 19]. It belongs to a broad line of work that infers carrier-side behavior from the device side. Designed probes can fit a small model: Qian et al. recovered UMTS RRC state machines and timers [55]. Configurations the network signals to the UE can be crawled [47]. Paired, controlled replays can detect hidden policies statistically: Wehe drew on 1,045,413 tests across 2,735 ISPs [56]. Network tomography infers internal network characteristics from end-to-end measurements [114] and states exact conditions under which link metrics are identifiable from path measurements [115]. Active-probing diagnosis chooses probes to localize faults in distributed systems [116] and adapts probe selection as evidence arrives [117]. This last line is the natural formal basis for the whiteboard's "design experiments for system inference and diagnosing failures".

In all of this work **the instrument is treated as transparent**: the inferred policy is computed from values whose derivation was never recorded. The proposed change is a pre-step at (c), not a new loop. Before a field validates or invalidates a hypothesis, check that its passport's procedure can resolve the hypothesis's time scale and statistic. If it cannot, or if the procedure is unknown, the field is flagged and goes to C3 or C5. Network inference stays the target.

**Solved.** Inferring network parameters from cheap, repeatable probes; crawling signaled configuration; statistically admitted inference of coarse policies; identifiability theory for network tomography; information-driven probe selection.
**Open.** Checking the instrument inside the loop. We found no study that measures how carrier-policy conclusions change under alternative post-processing procedures.

##### II-B2.2 Learning closed implementations

Active automata learning builds state machines of black-box implementations and compares them across implementations. DIKEUE learns the LTE control-plane FSMs of commercial UEs and treats pairwise deviations as a proxy for noncompliance [57]. 5GBaseChecker scales this to 17 commercial 5G basebands and finds 22 issues [58]. These methods assume deterministic targets, cheap resets and a controllable peer, and they learn control structure over messages, not numeric derivation rules. Their differential-testing oracle carries over directly. It points to cross-vendor comparison of derived KPIs (for example, XCAP against TEMS Discovery [38]), which we found no work doing.

**Solved.** Black-box FSM learning of message-level implementations at commercial scale; oracle-free differential comparison.
**Open.** Targets that are numeric procedures; learning under costly, stateful queries.

##### II-B2.3 Reverse engineering closed artifacts: decoding vs meaning

Format reverse engineering recovers syntax and field types from sets of messages [59], and cellular payloads have open decoders [1, 2]. Recovering **what a closed post-processor does to produce a number** is much rarer. The closest precedent we found is outside wireless: Brønd et al. recovered the closed ActiGraph "counts" aggregation by pushing designed sinusoids through the vendor software [60]. NIST's Statistical Reference Datasets supply certified known answers for testing statistical software [103], which is a precedent for a known-answer lab. Signal forensics estimates hidden processing history from outputs alone [61]. We found no published reverse engineering of a commercial drive-test post-processor's per-parameter procedures.

**Solved.** Syntax-level format recovery; open payload decoding; recovery of one closed aggregation via known-answer inputs (outside wireless); certified reference datasets for software.
**Open.** Linking a decoded field to the procedure behind the derived value; build dependence; known-answer benchmarks for scoring procedure recovery.

##### II-B2.4 Documentation vs behavior, and oracle-free checks

Documentation-guided testing turns documented constraints into inputs and exposes mismatches between documentation and implementation [62]. Metamorphic testing replaces a missing oracle with relations between outputs [63]; for post-processors, examples are invariance to time shifts or time-zone shifts, and consistency when bins are merged. Oracle assessment measures how often a check misses planted faults or raises false alarms [113]. Our setting reverses the usual premise. The procedure is written nowhere public: in 12 of 15 open-book runs the model reported finding nothing usable. Where documentation exists, it departs from behavior on value-changing points.

**Solved.** Mining constraints from documentation; documentation-behavior fuzzing; metamorphic relations; oracle assessment by planted faults.
**Open.** Specifications that must come from experiments rather than text; a catalog of metamorphic relations for wireless KPI post-processing (we found none).

##### II-B2.5 Query-based recovery of hidden functions and discriminating experiments (the whiteboard's "experiment procedure determination")

Classical work selects experiments that discriminate among candidate models [64], with near-optimal criteria under noise [65]. Program synthesis uses distinguishing inputs [66], and screening identifies which options influence an output [67]. The closest formal match to our target is REGAL, which recovers group-by columns, aggregate functions and filters from a base table and its report [68]. Xpose extracts SQL hidden in an opaque executable by mutating databases, using LLM hints [69]. These methods assume cheap, deterministic, side-effect-free queries and exact outputs.

The real tool departs from those assumptions in measurable ways. It has 12,881 parameter-setting cells per build. There is no read-only run: every launch rewrites about 20 settings files (more than 1.6 MB), including a file whose name changes each time. Binning results persist as new parameters in the model. In a controlled test, five of seven export forms wrote 0 bytes while exiting normally. Restores exist, the most expensive being a VM snapshot of the licensed install, but they are costly, and a restore cannot be confirmed by checking whether the environment changed, because it always changes. Runtimes that make agent tool use transactional exist [105]. They settle effects the runtime can observe; this tool's internal state changes are not visible in its exports. These are technical challenges. The fundamental part is identifiability (Sec. 4, FP3).

**Solved.** Exact recovery of hidden functions within a stated language; information-theoretic query choice; transactional tool use where effects are observable.
**Open.** Recovery under costly, stateful, noisy queries with per-field identifiability; a hypothesis space for measurement procedures (grids, bin edges, statistics, event windows, clocks).

##### II-B2.6 LLM agents that learn opaque tools

Agents now learn tools by experiment. DRAFT rewrites tool documentation from trials [70], and PLAY2PROMPT optimizes tool instructions through tool play [112]. OpaqueToolsBench benchmarks underspecified tools [71]. ZhuLong infers undocumented behavior of a commercial EDA API by counterfactual exploration and reaches 78.5% Pass@1 against 23.6% for a plain LLM. Removing sandbox execution costs 41.2 points, while self-exploration adds only 3.2 [72]. BoxingGym finds current LLMs (GPT-4o, Claude 3.7 Sonnet, Qwen-2.5) weak both at designing experiments scored by expected information gain and at model discovery [73]. Belief revision has old and new machinery: truth maintenance re-labels dependents when a premise is withdrawn [106], and MemTX stages agent-memory writes as transactions whose retraction repairs derived records [107]. C5's claim store builds on both. Every tool-learning system we found judges learned knowledge by task success or by the agent's own examples. None that we found admits a learned tool fact only through a path independent of both the tool and the agent.

**Solved.** Operating under-documented tools and raising task success by trial and error; dependency-directed belief revision.
**Open.** Independent admission of learned facts; benchmarks with costly stateful runs and planted false documentation claims; scoring recovered procedures rather than completed tasks.

---

## 3. Cross-cutting: AI for scientific discovery, and how its claims are verified

### X.1 AI scientists and their audits

AI systems now run the research loop in computational sandboxes [74] and drive autonomous labs [75]. Independent audits keep locating the failure between the raw signal and the claimed value. A re-analysis of A-Lab concludes that no new materials were shown. It finds that about two thirds of the claimed successes are likely known compositionally disordered versions of the predicted ordered compounds, and that automated Rietveld analysis of powder XRD is "not yet reliable" [76]. The A-Lab paper itself is now registered as a synthesis of *inorganic* rather than *novel* materials. Audits of AI-scientist systems find leakage, metric misuse and post-hoc selection that trace logs reveal far better than papers [77]. For Kosmos, independent scientists judged 79.4% of report statements accurate, and 57.9% of synthesis statements [78]. Rigorous falsification with Type-I error control exists, but it assumes the data are trustworthy [79].

**Solved.** Automated research loops in sandboxes; statistically controlled falsification on trusted data.
**Open.** Certifying the numbers AI systems consume or produce. We found no AI-scientist system that records or tests the measurement procedure behind its inputs.

### X.2 Data-analysis agents and analytic flexibility

Data-driven discovery benchmarks are hard: the best system scores 25% on DiscoveryBench [80]. BLADE scores multiple defensible analyses against expert ground truth [81], and in Many Analysts, 29 teams reached odds ratios from 0.89 to 2.93 on one dataset [82]. ConditionalQA scores whether answers state their conditions, for text [83]. In all of these the input values are fixed: the benchmarks vary the analysis, not the measurement.

**Solved.** Executable scoring of analysis; analyst choice as a variable.
**Open.** Ground truth that varies the measurement procedure, with a test of condition disclosure for measurement data.

### X.3 Agents on professional software and instruments

Agents operate professional scientific software. On ScienceBoard, average success ranges from 0% to 15% across observation settings, and is about 20% overall for the strongest backbones [9]. Agents also operate real precision instruments through vendor GUIs [84]. An audit of 15 scientific tools finds 91 silent failures, where an invocation appears successful but information is missing [85]. This matches our finding that five of seven export forms wrote nothing while exiting normally. These benchmarks score the end state; none that we found scores whether the agent recovered, and could prove, what the software computes.

**Solved.** Operating closed professional software and instruments. We make no claim here.
**Open.** Recovering and proving the procedures inside such software.

### X.4 Verifying agent claims

LLMs cannot reliably self-correct their reasoning without external feedback [86]. Kambhampati et al. argue that LLMs cannot self-verify, and propose LLM-Modulo frameworks that pair them with external model-based verifiers [87]; this is a position paper. A second model is not independent: across 350+ LLMs on one leaderboard, models agree 60% of the time when both err [88]. Agents over-claim task completion by 28.7-37.9 points relative to official evaluators [89]. Software engineering learned long ago that separately built versions still fail together [90], and that coincident failure has to be modeled rather than assumed away [111]. Certification infrastructure exists for declared properties [99, 100].

**Solved.** Showing that self-correction fails without external feedback; measuring correlated errors and over-claiming; certifying declared properties.
**Open.** A sound verifier when the instrument itself is the only witness, and a **test** of its independence from both the tool and the agent.

### X.5 How other measurement sciences carry procedure metadata

Mature communities attach procedure metadata at generation time, because they own the pipeline. ESMValTool writes W3C-PROV records, with preprocessor settings and software versions, for every output [91]. The Digital Calibration Certificate lists the methods, influence conditions and equipment for each measurement result [92]. In network measurement, LMAP attaches the producing Task and Action options to every result [98], and 3GPP network counters declare their collection method and condition [93]. These are the best existing analogues of a per-value passport, and all of them are *emitted by the producer*. For XCAP-exported radio KPIs the producer cannot emit what the tool does not disclose, and the exports do not carry the configuration that produced them. So the procedure has to be *measured*.

**Solved.** Per-output provenance when the pipeline is owned; per-result method records in calibration and network measurement.
**Open.** Procedure content recovered from outside a closed pipeline and certified by an independent path.

---

## 4. Fundamental problems

The problems are consolidated to one science problem and three that block it. Engineering difficulties are listed separately.

- **FP0, the science problem.** Attributing application failures to wireless causes, and those causes to architecture decisions, from observational drive-test data. Each attribution is only as valid as the fields that validate it (Sec. 1).
- **FP1, the value-procedure gap, staged.** v = A_post(b; θ) ∘ C_chip ∘ F_L3(k) ∘ M_L1(ι) (II-A.1). The procedure is part of the measurand, which makes this definitional uncertainty [10]. The time base and event-counting rules are instances of θ.
- **FP2, knowledge of θ sits with no party that can certify it.** The producer does not know it. The vendor's declared rules are incomplete and depart from behavior. The tool's success signals are decoupled from correctness. An agent, or a second model, cannot certify its own findings.
- **FP3, identifiability of θ from outputs.** In the tomography sense [114, 115], only some procedure classes are separable from given exports. The honest output for the rest is a set of consistent procedures, or "not inferable".

**Rebuttal 1: "Why not re-export the held raw logs at the highest fidelity?"** This is the whiteboard's own practice, and for held logs it is part of the answer. It does not close the gap, for five reasons. (i) The finest setting fixes only the grid. The event-counting rule (37 vs 2,151) and the export time zone (the 5-hour offset) are not sampling settings, and the manual says sampling "is not applied for Call Events and Signaling Messages". (ii) Analyses need aggregates, and any aggregate the tool computes goes through per-parameter rules, 428 of 1,171 of which are blank in its own declared table. Doing all aggregation in open code is the right answer, and that is C2: it needs the independent decode path, which today covers payloads of 3 of at least 141 codes. (iii) The finest export is still downstream of L1 and, for reported quantities, of L3 (II-A.1). (iv) Public releases, including [20], went out at 1 s; other groups' raw logs are not held; and the build drifts (label 5.33.0.15, build paths 5.33.0.9). (v) "All data available" fails per field: packet capture was off in 8 traces (3.3 GB), stated nowhere.

**Rebuttal 2: "Why not ask the vendor?"** A vendor answer is welcome, and it would enter the passport as *declared* content. But it is still a declaration, and for one build. The tool's own declared table leaves 428 of 1,171 parameters blank, four documented points depart from behavior on values, the version label does not pin the binaries, and past corpora were produced by earlier builds.

**Technical challenges (real, but not fundamental).**
- **T1, costly and stateful queries.** There is no read-only run. Restores are costly and cannot be confirmed from the environment (II-B2.5). The install is licensed, so interaction is budgeted. Whether a VM-snapshot restore returns the licensed install to an identical state is untested **[to measure]**.
- **T2, time base and joins.** Radio-to-application alignment is an instance of FP1. Our best bound is ±1.5 s.
- **T3, coverage of the independent path.** Payloads are decoded for 3 of at least 141 codes, and DIAG compatibility is undetermined.
- **T4, scale.** There are 12,881 parameter-setting cells per build, and our own hand loop was misled three times.

**Why AI makes this urgent.** AI consumers and benchmarks compute or read gold answers from files whose procedure is taken as given. An answer can then be "correct" under the hidden setting and wrong under another. Models trained on pooled corpora can learn the processing signature itself. Agents will produce metadata at scale that must be admitted by something other than themselves. Our only evidence is small: none of 5 numeric answers named its setting (one model family, one grader, single seed).

---

## 5. Contributions we can highlight

Status is given as in the figure legend.

- **C0 Formal problem statement** (whiteboard contribution 1; dashed, proposed addition). A failure-attribution question is evaluated on fields defined by the staged value model. Each stage is classified as fixed by standard, recoverable from the log, post-processor-specific, or not identifiable. A fitness condition ties a field's procedure to the hypothesis's time scale.
- **C1 Value Passport** (dashed). Measured per-parameter, per-build procedure content: the TS 32.404 template fields [93] mapped into a Croissant 1.1 PROV-O profile, with a tier and a range.
- **C2 Two-Path Rule, scoped** (dashed; the reader is solid). A value is certified only through a path that shares nothing but raw log bytes with the tool and any agent, with disjointness traced and coincident failure measured. Year-1 minimum: the codes behind the headline KPIs, a DIAG-compatibility go/no-go, and a fallback. An optional known-answer tier (emulator-applied conditions) is future work.
- **C3 WiReady** (dashed). Field-level readiness with calibrated abstention; the question-to-field fitness check (whiteboard contribution 3); the condition-disclosure test; and a Year-1 audit of 15-20 public releases.
- **C4 Comparable existing corpora** (dashed). The five campaigns (raw logs held; confirm for the joint Chicago/Minneapolis campaign), re-derived or flagged, with ranges, join bounds, an event layer, audited labels and re-tested findings.
- **C5 Recovery Engine** (dotted, conditional). Discriminating experiments on the closed tool (whiteboard contribution 4, tool-side reading); it never certifies.
- **Framing: the instrument check inside the group's loop** (II-B2.1).

---

## 6. Synthesis: where the gap sits

| Layer | Solved by prior work | Open (we found none) | Our contribution |
|---|---|---|---|
| Driver | Cross-layer failure diagnosis with co-captured, synchronized data [121, 8]; configuration attribution [45, 19, 46] | Attribution on legacy corpora with a fitness test per validating field | **C0** formal problem statement; framing at step (c) |
| I · containers and definitions | Croissant 1.1 value-level PROV-O, SOSA, DQV, RFC 8911/8193, TS 32.404, MDT, TS 38.215/38.133 [6, 22, 101, 5, 98, 93, 94, 4, 95] | Measured procedure content for drive-test post-processing of UE diagnostic logs | **C1 Value Passport** (a profile, not a new format) |
| I · readiness and verification | Dataset-level scoring [26, 27, 109]; declared-property certification [99, 100]; Band B named [25] | Field-level Band B with calibrated abstention; a real denominator | **C3 WiReady** + Year-1 audit |
| I · release | Large corpora; campaign-level harmonization including sampling configuration [15] | Cross-group, cross-tool comparability with per-field provenance | **C4** |
| II-A · conduction | Open decoding [1, 2]; scheduler-side telemetry [41]; scanners and conducted tests for the physical quantity [14, 96] | An independent path for post-processed values; end-to-end calibration of logging plus post-processing | **C2 Two-Path Rule** (scoped) |
| II-B1 · data analysis | Cross-layer RCA [8]; prediction [46]; causal tuning [49]; spec retrieval and field semantics [124, 128, 129]; agentic measurement [119] | Processing as a factor; field selection by procedure fitness; value-level admission | **C3** fitness and disclosure; **C4** re-tested findings |
| II-B2 · reverse system | Network inference [47, 56, 55]; tomography identifiability [114, 115]; active probing [116, 117]; automata learning [58]; hidden-query recovery [68, 69] | Checking the instrument inside the loop, under costly stateful queries, with independent admission | **C3** (passive), **C5 Recovery Engine** (conditional) |
| AI for science | Agents operate software [9]; external feedback beats self-correction [86] | Certifying what closed instruments' numbers mean | **C2** admission rule; **C3** disclosure test |

## References

[1] Y. Li, C. Peng, Z. Yuan, J. Li, H. Deng, T. Wang. "MobileInsight: Extracting and Analyzing Cellular Network Information on Smartphones." ACM MobiCom 2016. https://doi.org/10.1145/2973750.2973751
[2] B. Hong, S. Park, H. Kim, D. Kim, H. Hong, H. Choi, J.-P. Seifert, S.-J. Lee, Y. Kim. "Peeking Over the Cellular Walled Gardens: A Method for Closed Network Diagnosis." IEEE TMC, 2018 (tool: SCAT, https://github.com/fgsect/scat). https://doi.org/10.1109/TMC.2018.2804913
[3] P1 Security. QCSuper. https://github.com/P1sec/QCSuper
[4] 3GPP TS 38.215 V18.4.0 (ETSI TS 138 215 V18.4.0, 2025-01). https://www.etsi.org/deliver/etsi_ts/138200_138299/138215/18.04.00_60/ts_138215v180400p.pdf
[5] M. Bagnulo, B. Claise, P. Eardley, A. Morton, A. Akhter. "Registry for Performance Metrics." IETF RFC 8911, 2021. https://www.rfc-editor.org/rfc/rfc8911.html
[6] MLCommons Croissant WG. Croissant Format Specification v1.1, Jan 29, 2026. https://docs.mlcommons.org/croissant/docs/croissant-spec-1.1.html
[7] T. Lebo, S. Sahoo, D. McGuinness (eds.). PROV-O. W3C Recommendation, 2013. https://www.w3.org/TR/prov-o/
[8] F. Yi, H. Wan, K. Jamieson, O. Michel. "Automated, Cross-Layer Root Cause Analysis of 5G Video-Conferencing Quality Degradation" (Domino). ACM IMC 2025. https://doi.org/10.1145/3730567.3764434
[9] Q. Sun, Z. Liu, C. Ma, Z. Ding, F. Xu, et al. "ScienceBoard." ICLR 2026. https://arxiv.org/abs/2505.19897
[10] JCGM 200:2012 (VIM), entry 2.27. https://www.bipm.org/en/committees/jc/jcgm/publications
[11] J. Yeo, D. Kotz, T. Henderson. "CRAWDAD." ACM SIGCOMM CCR 36(2), 2006. https://doi.org/10.1145/1129582.1129588
[12] J. Breen et al. "POWDER." ACM WiNTECH 2020. https://doi.org/10.1145/3411276.3412204
[13] A. Alkhateeb et al. "DeepSense 6G." IEEE Commun. Mag. 61(9), 2023. https://doi.org/10.1109/MCOM.006.2200730
[14] W. Wiedner, L. Eller, M. Mussbah, D. Rössler, V. Maresch, P. Svoboda, M. Rupp. "The Vienna 4G/5G Drive-Test Dataset." Scientific Data, 2026 (preprint arXiv:2603.02638; data https://doi.org/10.5281/zenodo.21372657). https://doi.org/10.1038/s41597-026-08259-z
[15] A. H. Fahim Raouf et al. "Curated Wireless Datasets for Aerial Network Research." arXiv:2510.08752v2, 2026. https://arxiv.org/abs/2510.08752
[16] A. Narayanan et al. "Lumos5G." ACM IMC 2020. https://doi.org/10.1145/3419394.3423629
[17] A. Narayanan et al. "A First Look at Commercial 5G Performance on Smartphones." WWW 2020. https://doi.org/10.1145/3366423.3380169
[18] A. Hassan et al. "Vivisecting Mobility Management in 5G Cellular Networks." ACM SIGCOMM 2022. https://doi.org/10.1145/3544216.3544217
[19] R. A. K. Fezeu et al. "Unveiling the 5G Mid-Band Landscape." ACM SIGCOMM 2024. https://doi.org/10.1145/3651890.3672269
[20] M. I. Rochman, W. Ye, Z.-L. Zhang, M. Ghosh. IEEE TCCN 11(3), 2025; dataset https://github.com/ndghoshlab/mpls-chi-2022-2024. https://doi.org/10.1109/TCCN.2025.3558062
[21] T. Gebru et al. "Datasheets for Datasets." CACM 64(12), 2021. https://arxiv.org/abs/1803.09010
[22] A. Haller et al. (eds.). SOSA/SSN. W3C Recommendation, 2017. https://www.w3.org/TR/vocab-ssn/
[23] ETSI TS 102 250-6 V1.3.1 (2019-11). https://www.etsi.org/deliver/etsi_ts/102200_102299/10225006/01.03.01_60/ts_10225006v010301p.pdf
[24] NSF 26-512. https://www.nsf.gov/funding/opportunities/ai-datasets-unlocking-dataset-value-ai-enabled-scientific-discovery/nsf26-512/solicitation
[25] N. D. Lawrence. "Data Readiness Levels." arXiv:1705.02245. https://arxiv.org/abs/1705.02245
[26] K. Hiniduma et al. "AIDRIN." SSDBM 2024. https://arxiv.org/abs/2406.19256
[27] S. R. Wilkinson, P. Shpilker, W. Brewer. "SetGo." SSDBM 2026. https://doi.org/10.1145/3828820.3828827
[28] K. P. Mok, K. C. Claffy. CANIS. NSF Award 2531134. https://www.nsf.gov/awardsearch/show-award/?AWD_ID=2531134
[29] M. Sana et al. TeleLogs. arXiv:2507.21974. https://arxiv.org/abs/2507.21974
[30] A. Feng et al. "TelecomTS." arXiv:2510.06063. https://arxiv.org/abs/2510.06063
[31] V. Paxson. "Strategies for Sound Internet Measurement." ACM IMC 2004. https://doi.org/10.1145/1028788.1028824
[32] K. MacMillan et al. "Ookla Speedtest vs NDT7." POMACS 7(1), 2023. https://doi.org/10.1145/3579448
[33] B. Kaplan, S. Keşir, A. F. Coşkun. arXiv:2504.16039, 2025 (preprint). https://arxiv.org/abs/2504.16039
[34] M. E. Alim, T. H. Morris. "From 'Available' to 'Reusable': ... Industrial Control System Datasets ..." IEEE SPW 2026. https://doi.org/10.1109/SPW72489.2026.00041
[35] A. Alkhateeb. "DeepMIMO." ITA 2019. https://arxiv.org/abs/1902.06435
[36] J. Hoydis et al. "Learning Radio Environments by Differentiable Ray Tracing." arXiv:2311.18558. https://arxiv.org/abs/2311.18558
[37] Accuver. XCAP product page. https://www.accuver.com/sub/products/view.php?idx=101
[38] Infovista. TEMS Discovery. https://www.infovista.com/products/tems-discovery/network-testing-analytics
[39] Y. Xie, K. Jamieson. "NG-Scope." POMACS 6(1), 2022. https://doi.org/10.1145/3508032
[40] Google. CellSignalStrengthNr. https://developer.android.com/reference/android/telephony/CellSignalStrengthNr
[41] H. Wan, X. Cao, A. Marder, K. Jamieson. "NR-Scope." ACM CoNEXT 2024. https://doi.org/10.1145/3680121.3697808
[42] U. Paul et al. ACM IMC 2022. https://doi.org/10.1145/3517745.3561441
[43] V. Paxson. SIGMETRICS 1998 (see also Moon, Skelly, Towsley, INFOCOM 1999). https://doi.org/10.1145/277851.277865
[44] J. T. Quimby, D. F. Williams, K. A. Remley, D. Ribeiro, R. Sun, J. Senic. "Millimeter-Wave Channel-Sounder Performance Verification Using Vector Network Analyzer in a Controlled RF Channel." IEEE TAP 69(11), 2021. https://doi.org/10.1109/TAP.2021.3083737
[45] A. Narayanan et al. "A Variegated Look at 5G in the Wild." ACM SIGCOMM 2021. https://doi.org/10.1145/3452296.3472923
[46] W. Ye et al. Prism5G. ACM SIGCOMM 2024. https://doi.org/10.1145/3651890.3672250
[47] H. Deng et al. MMLab. ACM IMC 2018. https://doi.org/10.1145/3278532.3278546
[48] Y. Liu, J. Huang, S. Fahmy, C. Peng. ACM IMC 2025. https://doi.org/10.1145/3730567.3764479
[49] L. Xue et al. CausalTune. ACM SIGCOMM 2026. https://doi.org/10.1145/3789240.3829109
[50] D. Wu et al. "NetLLM." ACM SIGCOMM 2024. https://doi.org/10.1145/3651890.3672268
[51] A. Fang et al. "OpenRCA 2.0: From Outcome Labels to Causal Process Supervision." arXiv:2606.27154 (work in progress). https://arxiv.org/abs/2606.27154
[52] S. Kapoor, A. Narayanan. Patterns 4(9), 2023. https://doi.org/10.1016/j.patter.2023.100804
[53] S. Steegen et al. "Multiverse Analysis." PPS 11(5), 2016. https://doi.org/10.1177/1745691616658637
[54] T. Zimmermann et al. "The More We Measure, The Less We See: On the Replicability and Repeatability of Mobile Starlink Measurements." ACM SIGCOMM 2026. https://doi.org/10.1145/3789240.3827586
[55] F. Qian et al. ACM IMC 2010. https://doi.org/10.1145/1879141.1879159
[56] F. Li et al. Wehe. ACM SIGCOMM 2019. https://doi.org/10.1145/3341302.3342092
[57] S. R. Hussain et al. DIKEUE. ACM CCS 2021. https://doi.org/10.1145/3460120.3485388
[58] K. Tu et al. 5GBaseChecker. USENIX Security 2024. https://www.usenix.org/conference/usenixsecurity24/presentation/tu
[59] J. Chandler, A. Wick, K. Fisher. BinaryInferno. NDSS 2023. https://doi.org/10.14722/ndss.2023.23131
[60] J. C. Brønd et al. MSSE 49(11), 2017. https://doi.org/10.1249/MSS.0000000000001344
[61] M. C. Stamm, M. Wu, K. J. R. Liu. IEEE Access 1, 2013. https://doi.org/10.1109/ACCESS.2013.2260814
[62] D. Xie et al. DocTer. ISSTA 2022. https://doi.org/10.1145/3533767.3534220
[63] S. Segura et al. IEEE TSE 42(9), 2016. https://doi.org/10.1109/TSE.2016.2532875
[64] G. E. P. Box, W. J. Hill. Technometrics 9(1), 1967. https://doi.org/10.1080/00401706.1967.10490441
[65] D. Golovin, A. Krause, D. Ray. NIPS 2010. https://arxiv.org/abs/1010.3091
[66] S. Jha et al. ICSE 2010. https://doi.org/10.1145/1806799.1806833
[67] N. Siegmund et al. ESEC/FSE 2015. https://doi.org/10.1145/2786805.2786845
[68] W. C. Tan et al. REGAL. PVLDB 10(11), 2017. https://doi.org/10.14778/3137628.3137648
[69] A. Pradhan, J. Haritsa. Xpose. arXiv:2504.10898. https://arxiv.org/abs/2504.10898
[70] C. Qu et al. DRAFT. ICLR 2025. https://arxiv.org/abs/2410.08197
[71] S. Hallinan et al. OpaqueToolsBench. arXiv:2602.15197. https://arxiv.org/abs/2602.15197
[72] Y. Liu et al. ZhuLong. arXiv:2608.07925. https://arxiv.org/abs/2608.07925
[73] K. Gandhi et al. BoxingGym. arXiv:2501.01540. https://arxiv.org/abs/2501.01540
[74] C. Lu et al. "The AI Scientist." arXiv:2408.06292. https://arxiv.org/abs/2408.06292
[75] N. J. Szymanski et al. "An Autonomous Laboratory for the Accelerated Synthesis of Inorganic Materials." Nature 624:86-91, 2023. https://doi.org/10.1038/s41586-023-06734-w
[76] J. Leeman et al. PRX Energy 3, 011002, 2024. https://doi.org/10.1103/PRXEnergy.3.011002
[77] Z. Luo, A. Kasirzadeh, N. B. Shah. arXiv:2509.08713. https://arxiv.org/abs/2509.08713
[78] L. Mitchener et al. "Kosmos: An AI Scientist for Autonomous Discovery." arXiv:2511.02824. https://arxiv.org/abs/2511.02824
[79] K. Huang, Y. Jin, R. Li, M. Y. Li, E. Candès, J. Leskovec. POPPER. arXiv:2502.09858. https://arxiv.org/abs/2502.09858
[80] B. P. Majumder et al. DiscoveryBench. arXiv:2407.01725. https://arxiv.org/abs/2407.01725
[81] K. Gu et al. BLADE. Findings of EMNLP 2024. https://arxiv.org/abs/2408.09667
[82] R. Silberzahn et al. AMPPS 1(3), 2018. https://doi.org/10.1177/2515245917747646
[83] H. Sun, W. W. Cohen, R. Salakhutdinov. ConditionalQA. ACL 2022. https://doi.org/10.18653/v1/2022.acl-long.253
[84] H. Deng et al. Owl-AuraID 1.0. arXiv:2603.29828. https://arxiv.org/abs/2603.29828
[85] S. Gopalan, D. Singh, S. Narayanan. arXiv:2609.26836. https://arxiv.org/abs/2609.26836
[86] J. Huang et al. ICLR 2024. https://arxiv.org/abs/2310.01798
[87] S. Kambhampati et al. LLM-Modulo (position paper). ICML 2024. https://arxiv.org/abs/2402.01817
[88] E. Kim, A. Garg, K. Peng, N. Garg. ICML 2025. https://arxiv.org/abs/2506.07962
[89] H. Li et al. "Who Holds the Pen?" arXiv:2609.29921. https://arxiv.org/abs/2609.29921
[90] J. C. Knight, N. G. Leveson. IEEE TSE SE-12(1), 1986. https://doi.org/10.1109/TSE.1986.6312924
[91] M. Righi et al. ESMValTool v2.0. GMD 13, 2020. https://doi.org/10.5194/gmd-13-1179-2020
[92] S. Hackel et al. "The Digital Calibration Certificate." PTB-Mitteilungen 127(4), 2017. https://doi.org/10.7795/310.20170403
[93] 3GPP TS 32.404 V19.0.0 (ETSI TS 132 404 V19.0.0, 2025-10). https://www.etsi.org/deliver/etsi_ts/132400_132499/132404/19.00.00_60/ts_132404v190000p.pdf
[94] 3GPP TS 37.320 V19.3.0, MDT (ETSI TS 137 320 V19.3.0, 2026-08). https://www.etsi.org/deliver/etsi_ts/137300_137399/137320/19.03.00_60/ts_137320v190300p.pdf
[95] 3GPP TS 38.133, NR; Requirements for support of radio resource management. https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3204
[96] 3GPP TS 38.533, NR; UE conformance specification; RRM. https://www.etsi.org/deliver/etsi_ts/138500_138599/138533/
[97] 3GPP TS 38.331, NR; RRC protocol specification, Sec. 5.5.3.2. https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3197
[98] T. Burbridge, P. Eardley, M. Bagnulo, J. Schoenwaelder. "Information Model for Large-Scale Measurement Platforms (LMAPs)." IETF RFC 8193, 2017. https://www.rfc-editor.org/rfc/rfc8193.html
[99] C. Cao (PI), T. K. Samuel, G. Rogers. CERTIFY. NSF Award 2613350, 2026-2029. https://www.nsf.gov/awardsearch/show-award/?AWD_ID=2613350
[100] SLSA. Verification Summary Attestation, SLSA v1.0. https://slsa.dev/spec/v1.0/verification_summary
[101] R. Albertoni, A. Isaac (eds.). Data Quality Vocabulary. W3C WG Note, 2016. https://www.w3.org/TR/vocab-dqv/
[102] SigMF: The Signal Metadata Format. https://github.com/sigmf/SigMF
[103] NIST ITL. Statistical Reference Datasets (StRD). https://www.itl.nist.gov/div898/strd/
[104] Y. C. Zhang et al. "SyncWISE." Proc. ACM IMWUT 4(3), 2020. https://doi.org/10.1145/3411824
[105] B. Mohammadi et al. "Atomix." arXiv:2602.14849. https://arxiv.org/abs/2602.14849
[106] J. Doyle. "A Truth Maintenance System." Artificial Intelligence 12(3), 1979. https://doi.org/10.1016/0004-3702(79)90008-0
[107] X. Li et al. "MemTX." arXiv:2607.23929. https://arxiv.org/abs/2607.23929
[108] M. D. Wilkinson et al. "The FAIR Guiding Principles." Scientific Data 3, 2016. https://doi.org/10.1038/sdata.2016.18
[109] A. Devaraju, R. Huber. F-UJI. Patterns 2(11), 2021. https://doi.org/10.1016/j.patter.2021.100370
[110] C. G. Northcutt, A. Athalye, J. Mueller. NeurIPS D&B 2021. https://arxiv.org/abs/2103.14749
[111] D. E. Eckhardt, L. D. Lee. IEEE TSE SE-11(12), 1985. https://doi.org/10.1109/TSE.1985.231895
[112] W. Fang et al. "PLAY2PROMPT." Findings of ACL 2025. https://arxiv.org/abs/2503.14432
[113] G. Jahangirova, D. Clark, M. Harman, P. Tonella. "Test Oracle Assessment and Improvement." ISSTA 2016. https://doi.org/10.1145/2931037.2931062
[114] M. Coates, A. Hero, R. Nowak, B. Yu. "Internet Tomography." IEEE SPM 19(3), 2002. https://doi.org/10.1109/79.998081
[115] L. Ma, T. He, K. K. Leung, A. Swami, D. Towsley. ACM IMC 2013. https://doi.org/10.1145/2504730.2504738
[116] I. Rish et al. IEEE/IFIP NOMS 2004. https://doi.org/10.1109/NOMS.2004.1317650
[117] I. Rish et al. "Adaptive Diagnosis in Distributed Systems." IEEE TNN 16(5), 2005. https://doi.org/10.1109/TNN.2005.853423
[118] A. Ramanathan, E. Kang, D. Han, S. Abdu Jyothi. ArachNet. arXiv:2511.10611. https://arxiv.org/abs/2511.10611
[119] A. Ramanathan, E. Kang, D. Han, S. Abdu Jyothi. "Airavat." arXiv:2602.20924. https://arxiv.org/abs/2602.20924
[120] R. Beltiukov et al. arXiv:2505.19305. https://arxiv.org/abs/2505.19305
[121] Q. A. Chen et al. "QoE Doctor." ACM IMC 2014. https://doi.org/10.1145/2663716.2663726
[122] Y. Li et al. "A Nationwide Study on Cellular Reliability." ACM SIGCOMM 2021. https://doi.org/10.1145/3452296.3472908
[123] Ö. Alay et al. MONROE. ACM MobiCom 2017. https://doi.org/10.1145/3117811.3117812
[124] A.-L. Bornea et al. "Telco-RAG." IEEE Globecom 2024. https://arxiv.org/abs/2404.15939
[125] R. Nikbakht, M. Benzaghta, G. Geraci. "TSpec-LLM." arXiv:2406.01768. https://arxiv.org/abs/2406.01768
[126] I. Karim, K. S. Mubasshir, M. M. Rahman, E. Bertino. "SPEC5G." arXiv:2301.09201. https://arxiv.org/abs/2301.09201
[127] M. M. Rahman, I. Karim, E. Bertino. "CellularLint." USENIX Security 2024. https://www.usenix.org/conference/usenixsecurity24/presentation/rahman
[128] Q. Huang, X. Wang, W. Guo, G. Gu, H. Hu. ConSeT. USENIX Security 2026. https://arxiv.org/abs/2604.04283
[129] Y. Suhara et al. Doduo. SIGMOD 2022. https://doi.org/10.1145/3514221.3517906



---

## Appendix A. Fundamental problems in detail

### FP0

FP0. The science problem, from the whiteboard: attributing application failures to wireless causes, and those causes to architecture decisions (NSA vs SA, band, TDD pattern, carrier aggregation), from observational drive-test data. Failures are identified from outside or application data. Causes live in radio and signaling fields. Each attribution is only as valid as the fields that validate it.

**Why fundamental.** The data are observational. An attribution therefore rests on joining streams recorded by different instruments at different time scales, and on fields whose values depend on how they were produced. A failure lasting about two seconds cannot be validated by a field whose procedure averages over one second under an unknown clock offset, however good the analysis. Stronger analysis or stronger models do not remove this; only knowing and checking each field's procedure does (FP1-FP3).

**Where it bites.** II-B1: which fields to extract for a question, and validating hypotheses. II-B2: step (c) of the group's loop. And every AI model trained to predict application failures from radio data (C4 part 5).

**Closest attempts.** QoE Doctor diagnoses app QoE with automated UI control and cross-layer analysis down to RRC [121]. A nationwide study covers cellular failures on Android phones [122]. Domino traces 24 causal chains using NR-Scope telemetry and NTP-synchronized hosts [8]. Our own configuration-level attributions [45, 19, 46, 20]. ON-OFF loops traced to inconsistent triggers [48]. Speed-test contextualization [42]. All of these either capture the data together with synchronized clocks, or treat derived fields as observations.

**Our evidence.** Radio-application alignment holds within ±1.5 s for streaming and only to seconds for teleoperation, comparable to the 2.0 s median stall, so stall labels hold only at second level. 1-second averages hide 10-ms SINR dips: worst SINR is -20.0 vs -12.9 dB on one trace. 602 video stalls and 73,714 frames were recorded outside the tool. Packet capture was off in 8 traces (3.3 GB), stated nowhere, so some failure questions lack their application side.

### FP1

FP1. The value-procedure gap (definitional uncertainty), staged. A drive-test KPI is v = A_post(b; θ), with b = C_chip(F_L3(M_L1(x; ι); k); λ). The stages are: UE L1 measurement, with implementation choices ι; the RRC Layer-3 filter, with a network-signaled coefficient k; chipset logging λ; and the post-processor's procedure θ (sampling grid, per-parameter binning rule and statistic, records-versus-events rule, export time zone and clock, build). Decoding b correctly does not fix what v means. The time base and the event-counting rules are instances of θ.

**Why fundamental.** The procedure is part of the measurand, so the spread over admissible procedures is definitional uncertainty [10], and a better decoder or model does not reduce it. The stages differ in kind. TS 38.215 defines the quantity and leaves parts 'up to the UE implementation' [4], and TS 38.133 bounds accuracy and measurement period [95]. k is signaled in RRC messages [97], so it is recoverable from the log when those messages are logged. θ is post-processor-specific and unrecorded. Network counters declare collection method and trigger condition per measurement (TS 32.404 [93]), and MDT makes the logging interval and triggers configured parameters [94]. For drive-test post-processing of UE diagnostic logs we found no counterpart. Averaged exports cannot be un-averaged, so existing corpora, which NSF 26-512 targets [24], cannot be rescued by recording settings in the future. Re-exporting held raw logs at the finest setting does not close the gap either. It fixes only the grid. The event rule (37 vs 2,151) and the export time zone are not sampling settings. The tool's manual says sampling is not applied to call events and signaling messages. Any coarser aggregate the tool computes goes through per-parameter rules, 428 of 1,171 of which are blank in its own table. The finest export is still downstream of L1 and L3. And public releases, including [20], went out at 1 s, while other groups' raw logs are not held. Doing all aggregation in open code is the right answer, and that is C2.

**Where it bites.** Every layer. Layer I: releases state at most one dataset-level rule. II-B1: pooled training and cross-study comparison cannot tell a network effect from a processing choice, and sub-second questions are answered from 1-second aggregates. II-B2: network-policy inference inherits the unrecorded θ. AI consumers ingest bare numbers as facts.

**Closest attempts.** Containers and hooks: Croissant 1.1 value-level PROV-O [6], PROV-O prov:Plan [7], SOSA sosa:Procedure [22], RFC 8911 [5] and RFC 8193 per-result options [98]. Declared per-measurement rules for network counters [93] and MDT [94]. UE accuracy specified and conformance-tested [95, 96]. Tool dependence documented but not attached per value: AERPAW [15], Kaplan et al. [33], Ookla vs NDT7 [32]. Dataset-level prose: the Chicago/Minneapolis README [20]; Vienna omits its export configurations but records UTC timestamps [14].

**Our evidence.** Worst SINR is -20.0 dB at the finest setting and -12.9 dB at the documented 1-second default on one trace (fivefold in linear terms; aggregation arithmetic, not a defect). Counting rows turns 37 handover attempts into 2,151, while message pairing pairs 62 of 62. Export times sit 5 hours behind chipset time in two collections and match it in a third, most plausibly an unrecorded export time zone. None of five public releases checked (three ours) records its full export configuration. One build (XCAP-M 5.33.0.15) and pilot data.

### FP2

FP2. Knowledge of θ sits with no party that can certify it. The dataset producer does not know the procedure. The vendor's declared rules are incomplete, and its documentation departs from behavior on value-changing points. The tool's success signals are decoupled from correctness. An AI agent, or a second model, cannot certify its own findings. So the metadata has to be measured, carried with a tier and a range, and admitted only through a path whose independence from both the tool and the agent is measured.

**Why fundamental.** Producer-side documentation cannot contain knowledge the producer lacks. Datasheets, Croissant-RAI fields and readiness scores are filled in from what the producer supplies [21, 26, 27], and Band B is named but not automated [25]. 'Why not ask the vendor?' A vendor answer is welcome and would enter the passport as declared content. But it is still a declaration, and for one build. The tool's own table leaves 428 of 1,171 parameters blank, four documented points depart from behavior, the version label does not pin the binaries, and past corpora were produced by earlier builds. On the agent side, self-correction fails without external feedback [86], LLM errors are correlated across providers [88], and separately built versions still fail together [90, 111]. Silent failures are documented in other scientific tools [85]. Certification infrastructure [99, 100, 101] certifies declared properties of declared pipelines; here the property itself is undeclared. Scope limit: for UE-internal quantities the chipset log is the only primary record of the value this UE computed. Certification can therefore be independent of the post-processor and of any agent, but not of the chipset.

**Where it bites.** Layer I: passport tiers and readiness audits of legacy exports. II-A: building the independent decode path. II-B2: admitting what the reverse loop or C5 recovers. AI for science: agent-produced metadata.

**Closest attempts.** Producer-emitted per-output provenance where the pipeline is owned: ESMValTool [91], the Digital Calibration Certificate [92], LMAP result options [98], TS 32.404 counter templates [93]. Certification of declared properties: CERTIFY [99], SLSA verification summaries [100], DQV certificates [101]. Independent references: Paxson's consistency checks [31], open decoders [1, 2, 3], over-the-air telemetry for scheduler fields [41, 39], scanners and conducted tests for the physical quantity [14, 96], and injected-signal fidelity checks [28]. Known-answer recovery of a closed aggregation outside wireless [60]. Admission by specification rather than by the agent [89].

**Our evidence.** Declared rules cover 743 of 1,171 parameters (428 blank). Four documentation-behavior gaps bear on values (one seen in a single bin). In 12 of 15 open-book runs the model reported finding nothing usable in public material. The version label reads 5.33.0.15 while the build paths read 5.33.0.9. Five of seven export forms wrote 0 bytes with a normal exit. Our container reader counts 165 of 167 log codes on one ping trace but decodes payloads for only 3 of at least 141, and its matches have not passed an independence audit. The 50-of-50 agreement checks only the export step. 0 values are certified.

### FP3

FP3. Identifiability of θ from outputs. Many procedures produce the same export, so recovering θ is an inverse problem in the tomography sense. Only some procedure classes are separable from a given set of exports. Designed re-exports of held raw logs separate more. For the rest, the honest answer is a set of consistent procedures, or 'not inferable'.

**Why fundamental.** This is a property of the data, not of the solver. Network tomography gives exact conditions under which link metrics are identifiable from end-to-end measurements [114, 115], and the same question arises for θ. Instance-equivalence means a procedure can match every observed export and still be wrong elsewhere. Model-discrimination and active-learning criteria [64, 65], active-probing diagnosis [116, 117], program synthesis [66] and hidden-query recovery [68, 69] say how to choose queries; they cannot separate procedures that no available query distinguishes. For export-only corpora, averaging has already removed information. The cost and statefulness of queries to the real tool (no read-only run, costly restores) come on top of this. They are technical challenges, not the fundamental part.

**Where it bites.** C3 must abstain when rules are not identifiable. C5 must declare rules non-identifiable instead of guessing. C4 must flag corpora 'not comparable'. II-B2 network inference inherits whatever θ leaves unresolved.

**Closest attempts.** Tomography identifiability [114, 115]. REGAL's recovery of grouping, aggregate and filter from a table and its report [68]. Xpose [69]. Box-Hill discrimination [64]. Noisy active learning [65]. Active-probing diagnosis [116, 117]. Known-answer recovery of ActiGraph counts [60]. Signal forensics [61]. Performance-influence screening [67].

**Our evidence.** 12,881 parameter-setting cells per build and tool. 3 of 11 settings swept for procedure measurement so far (RQ1). Our own hand loop was misled three times. No identifiability analysis has been run yet; C3 and C5 are designed to report non-identifiable rules rather than hide them.


## Appendix B. Contributions we can highlight, in detail

### C0 Formal problem statement (whiteboard contribution 1; proposed addition to the

C0 Formal problem statement (whiteboard contribution 1; proposed addition to the PD, which currently folds it into the thesis paragraph). A failure-attribution question (an application failure from outside data, a candidate wireless cause, an architecture decision) is evaluated on fields defined by the staged value model v = A_post(b; θ), with b = C_chip(F_L3(M_L1(x; ι); k); λ). Each stage is classified as fixed by standard, recoverable from the log, post-processor-specific, or not identifiable. A fitness condition says a field can validate a hypothesis only if its procedure's time scale and statistic can resolve the hypothesized event. Definitional uncertainty is reported as the range of v over the procedures consistent with the record.

- **Layer:** Driver and II-B (frames Layer I's C1 and C3)
- **Novelty vs prior work:** Tomography formalizes the identifiability of network parameters from end-to-end measurements [114, 115]. Metrology defines definitional uncertainty [10]. 3GPP formalizes network counters [93] and UE measurement accuracy [95], and RFC 8911/8193 formalize IP-metric methods and per-result options [5, 98]. We found no formalization of a drive-test KPI as a staged function with per-stage identifiability classes, or of a field-fitness condition for failure attribution. This is also where the title 'Metrology for Wireless Data' sits: comparison to reference and registry-style definitions, brought to the step from UE diagnostic log to post-processed KPI.
- **Evidence / status:** Conceptual; to be written. It is supported by the FP0-FP3 evidence on one build: -20.0 vs -12.9 dB, 37 vs 2,151, the 5-hour offset, ±1.5 s alignment against a 2.0 s median stall. Sensitivity ranges exist for 2 of 11 settings on 1 trace (RQ7).
- **Maps to:** The thesis paragraph ('decoding is solved, meaning is not'); frames RQ1, RQ5 and RQ7; whiteboard contribution (1)

### C1 Value Passport: for each exported parameter, setting and tool build, a measur

C1 Value Passport: for each exported parameter, setting and tool build, a measured procedure record (instrument; sampling grid, averaging rule, statistic, records per event; time base; blind spots; sensitivity range; verification tier with x-of-n checked; date), linked to every value in that field. It is built as a measured instance of the TS 32.404 template fields (Description, Collection Method, Condition, Measurement Result/Units), mapped into a Croissant 1.1 PROV-O profile.

- **Layer:** I (describe), filled with content recovered by II-B2 (C3, C5) and admitted by C2
- **Novelty vs prior work:** The containers exist (Croissant 1.1 value-level PROV-O [6], PROV-O [7], SOSA [22], DQV [101], RFC 8911 [5], RFC 8193 [98]). Network counters declare per-measurement collection methods [93], and MDT configures UE logging intervals [94]. Domains that own their pipeline emit such records [91, 92]. Releases state at most a dataset-level rule [20, 14]. The new part is measured procedure content for drive-test post-processing of UE diagnostic logs, per parameter and per build, carrying a tier and a range rather than one asserted value. We found this nowhere. The novelty is the content, not the format.
- **Evidence / status:** By hand: declared rules cover 743 of 1,171 parameters (428 blank); four documentation-behavior gaps bear on values; a 5-hour export offset in two collections. Example passport: 'All' setting, minimum, export time zone unrecorded, range -20.0 to -12.9 dB, provisional. RQ1 today: no cell has a measured procedure yet, and 3 of 11 settings have been swept for procedure measurement.
- **Maps to:** C1 (RQ1); the target schema for synopsis item (1)

### C2 Two-Path Rule, scoped

C2 Two-Path Rule, scoped. A value is certified only when a path that avoids both the vendor tool and any AI agent reproduces it; otherwise it is provisional or unverified. Disjointness is established by provenance tracing (the paths share only raw log bytes), and coincident failure is measured with planted faults. Year-1 minimum: (1) the log codes behind the headline KPIs (SS-SINR/RSRP series and handover RRC messages; count to be set); (2) a go/no-go test of whether .drm payloads are standard DIAG packets that SCAT, MobileInsight or QCSuper parse; (3) the fallback if not, extending our reader's own payload decoding; (4) an independence audit of the reader's already-matched codes before any is claimed. Optional future tier: known conditions applied by a base-station emulator (TS 38.533-style), logged by XCAL and exported by XCAP, would certify the whole chain end to end. This is out of scope, because the proposal collects no new measurements.

- **Layer:** I (verification tiers), at the II-A / II-B2 boundary
- **Novelty vs prior work:** Verification tiers and certification exist as concepts: CERTIFY's machine-checkable proofs of declared integrity and provenance requirements [99], SLSA verification summaries [100], DQV certificates [101]. So do cross-checking principles [31], open decoders [1, 2, 3], methodology verification for agentic measurement workflows [119], and injected-signal fidelity checks [28]. Independence is known to fail when assumed [90, 111, 88]. The new part: tiers whose certifying path is tested for disjointness from both the vendor tool and the agent, with measured coincident failure, applied to post-processed radio KPIs. We found this in no wireless dataset. Scope, stated explicitly: for UE-internal fields the rule is independent of the post-processor and of agents, not of the chipset. For scheduler fields, over-the-air telemetry [41] can give a fully independent path.
- **Evidence / status:** Our container reader matches the tool's record counts for 165 of 167 log codes on one ping trace, but decodes payloads for only 3 of at least 141, and its matches have not passed the independence audit. The 50-of-50 value agreement checks only the export step. 0 values are certified. Five of seven export forms wrote 0 bytes with a normal exit. A GitHub API search found no public XCAL container reader (a narrow search). High risk until the DIAG go/no-go is run.
- **Maps to:** C2 (RQ4); synopsis item (2)

### C3 WiReady: an auditor that returns, for each field of a legacy export, the proc

C3 WiReady: an auditor that returns, for each field of a legacy export, the procedures consistent with it (sampling interval, averaging rule, clock offset, missing capture types, event-counting rule) at a stated coverage, or 'not inferable'. It is calibrated on re-exports at known settings and tested on held-out campaigns. Three proposed parts come with it: (a) a question-to-field fitness check (whiteboard contribution 3), which, for a stated failure question, ranks fields by whether their passport procedure can resolve the event, and so tests the whiteboard assumption 'highest fidelity, all data available' per field; (b) a condition-disclosure test of whether passports make an AI assistant state the setting its answer depends on, against a prompt-only baseline; (c) a Year-1 systematic audit of 15-20 public drive-test releases for tool, build, settings, time zone and raw-log availability.

- **Layer:** I (readiness), using passive II-B2 inference; the fitness check and disclosure test serve II-B1 and AI consumers
- **Novelty vs prior work:** Readiness and FAIR checkers work at dataset level [26, 27, 109]. Lawrence's Band B is a human checklist [25]. Signal forensics infers processing history in other domains [61]. Semantic retrieval over specifications and message fields exists [124, 125, 128, 129], and benchmarks score conditions in text [83] or task success [9, 80]. The new parts are per-field procedure inference for radio-KPI exports with calibrated abstention, field selection by procedure fitness, and a disclosure test for measurement data. We found none of them. C3 directly instantiates the solicitation's call for automated readiness platforms for legacy datasets [24].
- **Evidence / status:** Checks are manual today. Packet capture was off in 8 traces (3.3 GB), stated nowhere. Disclosure baseline: 0 of 5 numeric answers named their setting (one model family, one grader, single seed). If the prompt alone suffices, we report that passports add nothing there. The fitness check and the public-release audit are designed, not started.
- **Maps to:** C3 (RQ5, RQ6); synopsis items (1) and (2); whiteboard contributions (2) and (3)

### C4 Comparable existing corpora

C4 Comparable existing corpora. C1-C3 are applied to our five published campaigns: Lumos5G, 5Gophers, 5G mobility, 5G mid-band and Chicago/Minneapolis. Raw logs are held; this must be confirmed for the joint Chicago/Minneapolis campaign. The work has five parts: re-derive values under a stated procedure, or flag them 'not comparable'; give sensitivity ranges for headline KPIs; align radio and application data after the fact, stating a time bound for each join; build a deduplicated event layer with published counting rules; and derive application-grounded labels (for models that predict application failures from radio data) with a label-error audit. Then 2-3 published findings are re-tested with processing held constant.

- **Layer:** I (release), feeding II-B (re-tested findings, failure-prediction labels)
- **Novelty vs prior work:** AERPAW harmonizes campaign-level metadata, including sampling configuration, within one platform [15]. TEMS Discovery re-derives other vendors' logs under one undisclosed procedure [38]. Cross-tool comparisons [33, 32], after-the-fact stream synchronization [104], label-error auditing [110] and multiverse analysis [53] exist. The new part is open comparability with per-field provenance across groups and post-processors. We found no other radio-KPI corpora whose published findings can be re-tested with processing held constant or deliberately varied. Frame it as enhancing existing assets, not as finding errors.
- **Evidence / status:** Per our corpus inventory [to confirm], the corpus is two XCAL campaigns, one chipset-level and two app-log, which gives a cross-tool test inside our own data. 62 of 62 handovers pair under message pairing, against 37 vs 2,151 under row counting. Pilot: 602 video stalls and 73,714 frames recorded outside the tool; ±1.5 s streaming alignment. Nothing is harmonized yet, and expert labeling has not started.
- **Maps to:** C4 (RQ2, RQ3, RQ7, RQ8); synopsis item (3); whiteboard problem statement (a)

### C5 Recovery Engine (conditional; whiteboard contribution 4, tool-side reading)

C5 Recovery Engine (conditional; whiteboard contribution 4, tool-side reading). An AI system designs discriminating experiments on the closed tool over held logs: it chooses settings, modes and trace segments per license-hour, counts a run only if its effect clears the measured background churn, and may declare a rule non-identifiable. It hands every draft to C2 and never certifies. It adds an admission gateway, a known-answer lab with planted false documentation claims, an experiment-validity test, and a claim store that re-flags dependent fields when a claim is overturned.

- **Layer:** II-B2 (checking the instrument), cross-cutting AI for science
- **Novelty vs prior work:** Operating closed software is solved [9, 84], and transactional runtimes exist where effects are observable [105]. Agents learn undocumented tools by experiment [70, 112, 72, 71]. Agentic measurement workflows with methodology verification exist [118, 119]. Hidden-function recovery and experiment-selection criteria exist [68, 69, 64, 65, 67], as do active-probing diagnosis [116, 117] and dependency-directed belief revision [106, 107]. The new part: metadata about a closed instrument, admitted only through a path that shares nothing but raw log bytes with the tool and the agent, and scored by certified rules per person-hour and by false admissions, not by task success. For the algorithms A1-A3, only the application is new.
- **Evidence / status:** Designed, not built. Only the lab's synthetic containers and one measured churn footprint exist, and no agent has run the loop. There is no read-only run of the tool: every launch rewrites about 20 settings files (more than 1.6 MB). Restores, up to a VM snapshot, are costly and cannot be confirmed from the environment. Whether a snapshot restores the licensed install exactly is untested. C5 is gated by RQ9 against an analyst with scripts and a no-engine agent, with 0 admissions targeted among at least 30 planted false claims. If either baseline matches it, C5 is withdrawn and C1-C4 stand.
- **Maps to:** C5 (RQ9, RQ10); synopsis item (1) only

### Framing: an instrument check inside the group's own reverse loop

Framing: an instrument check inside the group's own reverse loop. The loop stays (a) measure application performance, (b) hypothesize about the system, (c) validate or invalidate with data. The new pre-step at (c): before a field validates or invalidates a hypothesis, check that it carries a passport whose procedure fits the hypothesis's time scale and statistic. If it does not, flag the field and route it to C3 (passive inference) or C5 (experiments). Network inference remains the loop's scientific target. It now runs on values that carry their procedure.

- **Layer:** II-B2, with the result fed back into Layer I
- **Novelty vs prior work:** Network-side reverse inference is established [55, 47, 56, 18, 46, 48], as are tomography and identifiability [114, 115], automata learning of closed implementations [57, 58] and format reverse engineering [59]. All of them treat the measurement instrument as transparent. We found no work that adds an instrument-fitness check to a network-inference loop, or that separates inference about a commercial wireless post-processor from certification. This framing organizes C0, C1, C3 and C5; it is not a separate deliverable.
- **Evidence / status:** Conceptual. It is supported by the FP0-FP3 evidence on one build, e.g., 10-ms dips hidden by 1-second bins, and stall labels that hold only at second level.
- **Maps to:** Links the thesis to the group's published method; C1, C3, C5; whiteboard '(2) Inferring System'


## Appendix C. Claims we must not make

- Do not claim a new metadata or provenance format. Croissant 1.1 already supports PROV-O down to individual values, and PROV-O, SOSA/SSN, DQV, SigMF, RFC 8911, RFC 8193 and the TS 32.404 template exist. Say 'a wireless-measurement profile of Croissant 1.1 PROV-O, built on the TS 32.404 template fields, with measured content'.
- Do not say that telecom standards lack per-parameter procedure metadata. TS 32.404 declares collection method and condition per network counter, MDT (TS 37.320) makes UE logging intervals and triggers configured parameters, TS 38.133/38.533 specify and test UE measurement accuracy, and RFC 8193 attaches task options to every result. Narrow every gap claim to 'drive-test post-processing of UE diagnostic logs'.
- Do not say that no independent reference exists for UE-internal quantities. Scanners and conducted emulator tests give references for the physical quantity. Only the specific value a field UE computed lacks one.
- Do not present the finest XCAP setting ('All') as raw data. It is still downstream of UE L1 measurement and, for reported quantities, of the network-configured L3 filter, and events and signaling are not sampled like metrics.
- Do not claim a decoding advance or the 'first open parser'. MobileInsight, SCAT and QCSuper decode Qualcomm and Samsung diagnostics. At most: 'we found no public reader for the XCAL container' (the GitHub search was narrow).
- Do not claim that we discovered KPIs depend on the tool or its settings. AERPAW, Kaplan et al. (a preprint), Ookla vs NDT7 and NG-Scope already show it, and the Chicago/Minneapolis README states XCAL's 1 s averaging and mode. Our claim is measuring the procedure per value on identical bytes and recovering it for existing corpora.
- Do not present -20.0 vs -12.9 dB as a vendor bug or a surprise. It is aggregation arithmetic, and the 1 s default is in the vendor's manual. Do not imply 1 s data are invalid: they suit coverage and long-run throughput, not sub-second questions.
- Do not say wireless data are 'locked inside vendor GUIs', or that the tool hides its configuration from itself. XCAP has CLI export, database push and APIs; the settings exist in tool state, and our assistant read the tool's model files. Say 'its exports do not carry the configuration that produced them'.
- Do not say the tool has 'no reset'. Restores exist, up to a VM snapshot. Say 'there is no read-only run, and restores are costly and cannot be confirmed from the environment'. Whether a snapshot restores the licensed install exactly is untested.
- Do not claim that verification tiers, certification of data pipelines or agentic measurement with a verifier are new concepts (SLSA VSA, DQV, CERTIFY, Airavat). The claim is certifying-path disjointness from both tool and agent, with measured coincident failure, for post-processed radio KPIs.
- Do not claim failure diagnosis from cross-layer mobile data, cross-layer root-cause analysis, throughput prediction as such, or cross-layer PHY-to-application analysis as new (QoE Doctor, Li et al. SIGCOMM'21, Domino, Lumos5G, Prism5G).
- Do not claim semantic retrieval over 3GPP specifications, LLM inference of RRC field dependencies, or semantic column annotation as new (Telco-RAG, TSpec-LLM, SPEC5G, CellularLint, ConSeT, Doduo). The claim is field selection by measurement-procedure fitness.
- Do not claim that retroactive stream alignment is unsolved in general (Paxson; Moon et al.; SyncWISE), or that cross-layer alignment is unsolved when designed in at capture. Only radio-to-application alignment in legacy drive-test corpora, with a stated bound per join, is open. Ours is ±1.5 s at best. Do not call the 5-hour offset a deep clock problem: it is most plausibly an unrecorded export time zone.
- Do not claim the first AI-ready, labeled, root-cause-labeled or multimodal wireless dataset (DeepSense 6G, TeleLogs, TelecomTS, Vienna 2026), or the first harmonization of wireless datasets (the AERPAW curated catalog harmonizes campaign metadata, including sampling configuration). Do not claim co-registered camera/LiDAR plus radio data.
- Do not claim an agent that operates closed or professional software, a general-purpose agent, or an agent benchmark (ScienceBoard, Owl-AuraID). C5 is judged only by the metadata it produces, and it never certifies.
- Do not claim that learning an undocumented tool by experiment, recovering hidden computations from queries, or dependency-directed belief revision is new (DRAFT, PLAY2PROMPT, ZhuLong, OpaqueToolsBench, REGAL, Xpose, TMS, MemTX). For A1-A3 only the application is new.
- Do not present 'scientific tools fail silently' or 'agents should not self-certify' as our findings (ToolUniverse audit, Who Holds the Pen, Huang et al., Kim et al., Knight and Leveson, Eckhardt and Lee). Cite them. Present LLM-Modulo as a position paper, not as empirical evidence.
- Do not claim that analysis choices changing results is a conceptual novelty (multiverse analysis, Many Analysts, BLADE). Present ours as the transfer of that idea to hidden instrument configuration.
- Do not state that agents are 'confidently wrong' or gullible. State only: 'none of 5 numeric answers named its setting (one model family, one grader, single seed)'. Do not count this result twice.
- Do not claim any value is certified or independently verified. Today 0 values are certified. The 50-of-50 agreement checks only the export step, payloads are decoded for 3 of at least 141 codes, and the reader's matched codes have not passed an independence audit.
- Do not claim the Two-Path Rule is independent of the chipset for UE-internal quantities (SS-SINR, RSRP).
- Do not say 'the tool misdescribes itself in 13 ways', and do not count the four XCAL collection product lines as four analysis tools. Cite only the value-changing documentation-behavior gaps (four), dated and tied to XCAP-M 5.33.0.15.
- Do not claim results across builds, vendors or domains. There is one installed build and no result outside wireless; say the pattern 'may transfer to other closed instrument software'.
- Do not state the per-campaign tool split, or that the group holds the joint Chicago/Minneapolis campaign's raw logs, as fact until the PI confirms. Keep them marked [to confirm].
- Do not describe the 15.3 GB parsed corpus as multi-workload: all 15 traces are ping. Do not use the numbers the vault excludes on purpose: 21.33% vs 17.00%, 165/166, '87% of handovers', 17.6 s, 310 handovers, Confident-Wrong candidates, P5 percentages, the open-book gain, restore timings and checkpoint tiers, 'binning is Sum', 'Area Binning forces Mode'.
- Do not quote Paxson (IMC 2004) verbatim until the wording is checked against the paper; paraphrase without quotation marks.
- Do not present Kaplan et al. as a peer-reviewed benchmark (it is a preprint), or OpenRCA 2.0 as telecom or network data (it covers microservice systems).
- Do not write 'no prior work', 'no standard covers X' or 'the first'. Write 'we found no', because the searches were limited (web-search quota exhausted, paywalled full texts).
- Do not frame the work as finding errors in community datasets (their authors may be reviewers), and do not claim partners, community adoption or MLCommons endorsement without letters. The Croissant profile is 'to be proposed'.
- Do not claim new measurements, and do not present network-side experiment design as a deliverable. All work re-derives, re-exports or audits logs already held; the 2020-24 campaigns are not re-run.

## Appendix D. Figure specification (two-layer process)

**Wireless science discovery on a data infrastructure: the group's loop, with an instrument check**

- **Problem statement (the driver)**
  - Diagnosing failures (ours, published (outlined): the group's research line): (a) identify app failures, which requires outside or app data; (b) find the wireless reasons for failures; (c) relate them to architecture decisions (NSA vs SA, band, TDD, CA). Formalized by C0 (proposed).
- **II Wireless science discovery: II-A Measurement conduction**
  - Conducting experiments (ours, published (outlined)): Commercial 4G/5G drive-test campaigns 2020-24: collectors (XCAL; phone apps) produce raw chipset diagnostic bytes, plus app data recorded outside the tool. Whiteboard: minimal set of params; designing experiments for system inference and diagnosing failures. Raw logs held for [N] of 5 campaigns [to confirm].
  - Closed post-processor (procedure θ) (prior work (grey): vendor): XCAP (also TEMS Discovery). Turns bytes into tabular KPIs and signaling messages under θ: sampling grid, per-parameter binning rule and statistic, records per event, export time zone and clock, build. Exports do not carry θ, and the chain upstream (UE L1, RRC L3 filter) remains even at the finest setting.
  - Independent decode path (ours, pilot evidence (solid)): Our XCAL container reader: 165 of 167 log codes counted like the tool on one ping trace; payloads for 3 of at least 141 codes; 15.3 GB at 283 MB/s. To be paired with open decoders (MobileInsight, SCAT, QCSuper) after a DIAG-compatibility go/no-go.
- **II Wireless science discovery: II-B Analysis**
  - II-B1 Data analysis (ours, published (outlined): current practice): Extraction into tabular KPIs plus signaling text messages, then semantic extraction with respect to the problem statement, then validate hypotheses. Entry assumption on the whiteboard: 'set to highest fidelity; all data available'.
  - II-B2 Reverse system loop (center) (ours, published (outlined): SIGCOMM'22, SIGCOMM'24): Inferring the system from measurements: (a) measure app performance, (b) make hypotheses about the system, (c) validate or invalidate with data, then back to (b). Target: carrier configuration and policy (handover, CA, mid-band deployment).
  - Instrument check at step (c) (ours, proposed (dashed)): Before a field validates a hypothesis: does its passport's procedure fit the hypothesis's time scale and statistic? If yes, proceed. If no or unknown, flag the field and route it to C3 or C5.
  - C5 Recovery Engine (ours, conditional (dotted)): Discriminating experiments on the closed tool over held logs; may declare rules non-identifiable; never certifies; gated by RQ9.
- **I Data infrastructure**
  - Existing containers and definitions (prior work (grey)): Archives and DOIs; Croissant 1.1 value-level PROV-O, SOSA, DQV; RFC 8911 and RFC 8193; TS 32.404 counter templates; MDT; TS 38.215/38.133/38.533. Slots and declared rules, but no measured drive-test procedure content.
  - C1 Value Passport with C2 tiers (ours, proposed (dashed)): Measured per-parameter, per-build procedure, range and tier (certified / provisional / unverified), linked to every value; TS 32.404 fields mapped into a Croissant 1.1 profile. Content enters only through C2. 0 values certified today.
  - C3 WiReady (ours, proposed (dashed)): Field-level readiness: set-valued procedures at stated coverage, or 'not inferable'; question-to-field fitness check; condition-disclosure test; Year-1 audit of 15-20 public releases. Tests the 'all data available' assumption per field (e.g., packet capture off in 8 traces).
  - C4 Comparable existing corpora (ours, proposed (dashed)): Five campaigns re-derived or flagged 'not comparable'; sensitivity ranges; join time bounds; deduplicated event layer; audited labels; 2-3 findings re-tested with processing held constant.

Flows:
- Problem statement -> II-B1 semantic extraction (the question decides which fields are extracted) and -> II-B2 step (a) (which app failures to measure)
- Conducting experiments -> raw chipset bytes + app data -> Closed post-processor (θ) -> tabular KPIs + signaling messages -> II-B1 Data analysis
- Raw chipset bytes -> Independent decode path -> independently re-derived values -> compared with the export in C2 -> tier recorded in C1
- II-B1 validate hypotheses -> II-B2 loop: (a) measure app performance -> (b) hypothesis about the system -> (c) validate/invalidate -> back to (b) (the figure's one loop)
- At step (c): Instrument check reads the C1 passport of each validating field; if it fits, the loop proceeds; if it is unfit or unknown, the field goes to C3 (passive inference) or C5 (experiments)
- C3 and C5 drafts -> C2 admission -> C1 (only tiered content enters a passport; an overturned claim re-flags dependent fields)
- Existing containers and definitions -> C1 is a profile of them (Croissant 1.1 PROV-O, TS 32.404 template fields)
- C1 + C2 + C3 -> C4 comparable existing corpora -> back to II-B (re-tested findings; failure-prediction labels; network inference on values that carry their procedure)
- Whiteboard assumption 'highest fidelity, all data available' -> checked per field by C3 and recorded by C1, instead of assumed

*Caption.* The group's research process, redrawn from the two whiteboards: a problem-statement driver (diagnose app failures, find the wireless reasons, relate them to architecture decisions) on top of two layers. In Wireless Science Discovery, measurement conduction (II-A) produces raw chipset bytes and app data, and a closed post-processor turns the bytes into tables and messages under a procedure θ that its exports do not carry. Analysis (II-B) has two parts: data analysis (extraction, semantic extraction with respect to the problem statement, hypothesis validation) and the group's reverse-system loop, which stays at the center with network inference as its target. The one new element in the loop is an instrument check at step (c). Data Infrastructure (I) supplies containers and standards (prior) and the proposed Value Passport with Two-Path tiers (C1/C2), WiReady (C3) and comparable existing corpora (C4). Legend: grey = prior work; outlined = ours, published; solid = ours, pilot evidence; dashed = ours, proposed; dotted = ours, conditional (C5). Everything shown as ours rests on one build (XCAP-M 5.33.0.15) and pilot data. 0 values are certified today, C5 is designed but not built, and the per-campaign tool split and raw-log holdings are to be confirmed. Method lists (REGAL, Xpose, automata learning, tomography, active probing) belong in a separate detail table, not in this figure.
