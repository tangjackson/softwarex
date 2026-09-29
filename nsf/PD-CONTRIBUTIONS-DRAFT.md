---
tags: [nsf, proposal, draft, project-description]
status: draft v1
created: 2026-09-28
solicitation: NSF 26-512 (AI Datasets)
section: Project Description · Contributions
---

> Draft v1 of the Contributions section (English). Pitch deck: https://claude.ai/artifact/UNkbxa5z1CEgpAnnb6UdEG · Related: [[NSF-PROBLEM-FORMULATION]] · [[NSF-26-512-FIT-AND-PLAN]]
> Body text ~1,540 words plus two tables. Bracketed [X] items are placeholders. The internal notes at the end are NOT for submission.

## Contributions

Wireless measurement datasets start as chipset diagnostic logs from drive tests; closed commercial software, a post-processor, turns them into key performance indicators (KPIs) such as SINR (signal quality) and handover counts. Decoding the logs is largely solved by open tools [21–23]; what the numbers mean is not. For AI this matters: a model trained on pooled corpora cannot tell a network effect from a processing choice. In our data, counting table rows instead of events turned 37 handover attempts into 2,151. On one trace, worst SINR is −20.0 dB at the tool's finest setting and −12.9 dB at its documented 1-second default, a fivefold gap in linear terms. Neither is wrong, since 1-second averages hide 10-ms dips, but none of five public releases whose top-level documentation we checked (three from our campaigns) records its full export configuration. Metrology calls this definitional uncertainty [1]. Existing corpora, this solicitation's focus [2], cannot simply add settings: exports are already averaged and documentation departs from behavior, so procedures must be measured by experiment and values re-derived where raw logs survive. All evidence comes from one build (XCAP-M 5.33.0.15, which post-processes XCAL logs) and our pilot data.

**C1. Value Passport.** For each exported parameter, setting and tool build, we measure the procedure once and link every value in that field to it: instrument, procedure (sampling grid, averaging rule, statistic, records per event), time base, blind spots (including choices 3GPP leaves "up to the UE implementation" [3]), sensitivity range, verification tier with how many values were checked of how many (C2), and date. For the SINR case the passport reads: 'All' setting, minimum, export time zone unrecorded, range −20.0 to −12.9 dB, provisional, 2026-09. The passport is a wireless-measurement profile of Croissant 1.1, which supports PROV-O provenance down to individual values [4, 5], so Croissant-aware pipelines can pool or refuse values by procedure. *New:* measured per-parameter procedure content for a closed post-processor, which we found nowhere: containers, registries and aggregation norms [4–9], tool-recording practice [10], known tool dependence [11, 12] and our test methods [13–15] exist, but releases state at most a dataset-level rule [34]. *Evidence (by hand):* the tool's declared rules cover 743 of 1,171 parameters (428 blank, meaning unknown); four documentation-behavior gaps bear on values (one seen in a single bin); export times sit 5 hours behind chipset time in two collections and match it in a third, recorded nowhere. *Deliverable:* the profile (to be proposed to the MLCommons Croissant working group), passports for every field of our XCAL-processed campaigns, and a dated per-build catalog of tool behavior.

**C2. Two-Path Rule.** A value is *certified* only when a path avoiding both the vendor tool and any AI agent reproduces it; otherwise it is *provisional* (one path) or *unverified*. Independence is tested: the paths share only the raw log bytes, with no decoder, intermediate file or code in common. Every step checks outputs, not exit codes: in a controlled test, five of seven export forms wrote 0 bytes while exiting normally. Fields drafted by our Recovery Engine (see Method) stay unverified until a certifying check passes; measured values are never agent-generated. *New:* tested independence and per-value tiers for a commercial post-processor's KPIs, which we found in no wireless dataset; assumed independence fails in N-version experiments [16], and attestations, quality vocabularies, verified pipelines for other data and open decoders exist [17–23]. *Evidence:* our reader for XCAL's container (we found no public one) counts the same records as the tool for 165 of 167 log codes on one ping trace but decodes payloads for only 3 of at least 141, and its value matches have not passed the independence test, so no value is certified. Our 50-of-50 value agreement compares the export with the tool's own tables, after we caught a table family that merely mirrored the export, so it checks only the export step. *Plan and deliverable:* pair our container reader with open payload decoders [21–23], starting with the codes behind headline KPIs; release tier terms within the passport profile, a check library and the reader or, if legal review blocks it, the reader's test outputs.

**C3. WiReady.** WiReady, an auditor we will build (its checks are manual today), instantiates for wireless the solicitation's "automated platforms that discern the readiness of legacy datasets at scale" [2]. It will infer how each field was produced (sampling interval, averaging rule, clock offset, missing capture types, event-counting rule), record this with tiers in Croissant's metadata and responsible-AI fields, and state limits such as "no application-layer throughput: packet capture was off." AI assistants and repositories can call it. We also test whether passports make an assistant state the setting its answer depends on, against a prompt-only baseline; if the prompt suffices, we report that passports add nothing here. *New:* per-field procedure inference and a disclosure test for measurement data, which we found nowhere; readiness and FAIR checkers work at dataset level [24–26], and benchmarks score answer conditions in documents or task success [27–29]. *Evidence (pilot):* by hand we found packet capture off in 8 traces (3.3 GB), stated nowhere. An AI assistant working from a shell, mostly by reading the tool's model files, solved 5 of 11 tool tasks fully and 5 partly, yet none of its 5 numeric answers named its setting; separately, 12 of 15 open-book runs reported no usable public material on the tool's behavior (one model family and grader; shell runs single-seed; numeric ground truth awaits calibration). *Deliverable:* WiReady and RQ5–RQ6 evaluation materials from held traces re-exported at known settings.

**C4. Comparable existing corpora.** We apply C1–C3 first to our five published campaigns (2020–24; one joint with the Ghosh lab), whose raw logs we hold: Lumos5G [30], 5Gophers [31], 5G mobility [32], 5G mid-band [33] and Chicago/Minneapolis [34]. Two [33, 34] used XCAL, one [32] chipset-level logs and two [30, 31] phone-side app logs, which gives a cross-tool test within our corpus (XCAL first, the rest in Year 2). They hold [X TB] of raw logs ([32] alone spans 6,200+ km on three US carriers) and have been cited [N] times; enhanced releases will be hosted at [repository], with privacy (device IDs, GPS) under the Data Management Plan. Partner XCAL or Keysight Nemo corpora follow where letters of collaboration exist. Five parts: (1) re-derivation from raw logs under a stated procedure or, for export-only corpora, procedure inference or a "not comparable" flag; (2) sensitivity ranges for headline KPIs; (3) after-the-fact radio-application alignment, each join stating its time bound; (4) a deduplicated event layer with published counting rules; (5) labels from application records, for models predicting application failures from radio data, with a label-error audit. *New:* open comparability with per-field provenance for existing radio-KPI corpora across groups and post-processors, which we found nowhere; closed tools re-derive other vendors' logs under one undisclosed procedure [35]; cross-tool comparisons [11, 12, 36] and each component method [22, 37–41] exist. *Evidence (pilot):* 602 video stalls and 73,714 frames recorded outside the tool; radio-application alignment within ±1.5 s for streaming and only seconds for teleoperation, comparable to the 2.0 s median stall, so stall labels hold only at second level. Nothing is harmonized yet; expert labeling has not started.

**Method: the Recovery Engine.** One build has 1,171 parameters × 11 settings = 12,881 cells, repeated per build and tool. Scripts run known sweeps; the engine, an AI agent, takes what they cannot: the 428 blank rules, documentation contradictions, control design and new builds. It treats documentation and version strings as hypotheses, runs sweeps and positive controls (checks first shown to catch a known behavior) on resettable tool snapshots, and drafts passport fields. It never certifies. Certification needs a human-reviewed check (known-answer, or a predicted output change for a controlled input change) that passes its positive control and runs on C2's agent-free, vendor-free path, with its author recorded. A second model cannot substitute: LLM errors correlate across providers [42], and self-correction without external feedback is unreliable [43]. Agents that learn opaque tools admit knowledge by task success, self-generated examples or their own sandbox runs [44–46]; certification independent of both agent and vendor tool, with that independence tested, is our difference. The engine is untested; we ran its loop by hand. In Year 1 (RQ1b) it is scored against scripts alone on rules a teammate establishes after it is frozen, and continues only at [X]% agreement with zero false certifications, including on planted false documentation claims (protocol in the Evaluation Plan); otherwise scripted sweeps fill C1 at smaller coverage, and C1–C4 stand without it.

**Measurable outcomes** (targets proposed).

| RQ | Measures | Today | Target |
|---|---|---|---|
| RQ1 (C1) | cells with verified procedure | none; 743 declared rules; 3 of 11 settings swept | [Year 1: X]; 12,881 on 2 builds by Year 2 |
| RQ1b (engine) | held-out agreement; false certifications; gain over scripts | loop run by hand | [≥X%]; 0 false certifications |
| RQ2 (C4.3) | time bound per join | ±1.5 s streaming; seconds teleoperation | a stated bound on every join |
| RQ3 (C4.4) | events matching message pairing and experts | 62/62 handovers paired | [X%] |
| RQ4 (C2) | headline-KPI values independently reproduced | 0; payloads 3 of ≥141 codes | [X%] by Year 2, else blocking codes named |
| RQ5 (C3) | inferred-field precision and recall at known settings | manual only | [≥X% precision] |
| RQ6 (C3) | numeric answers stating their setting | 0 of 5 | [≥Y%] with passports, above prompt-only |
| RQ7 (C4.2) | headline KPIs with a range | 1 trace, 2 of 11 settings | all 11 settings |
| RQ8 (C4) | corpora comparable; findings and model transfer re-tested | none | 5 corpora; [2–3] findings re-tested (e.g., [30, 32]); transfer before vs after |

**What we do not claim.** No new format, decoding advance or discovery that KPIs depend on the tool [11, 12]; no single method (each has precedent; our claim is their combination, which we found applied nowhere to closed wireless post-processors); no cross-layer root-cause analysis [47]; no first AI-ready or labeled wireless dataset [48–50]; no general-purpose agent or agent benchmark (RQ6 evaluates our metadata, not models). The SINR case is aggregation arithmetic, not a vendor defect; 1-second data suit coverage and long-run throughput, not sub-second questions such as dips, stalls and handover interruptions. We collect no new measurements; all work re-derives, re-exports or audits logs already held. The pattern may transfer to other closed instrument software [15]; we claim no results outside wireless.

**Mapping to the synopsis [2].**

| Synopsis item | Addressed by |
|---|---|
| (1) AI-based feature extraction, metadata generation, integration of multiple datasets | Recovery Engine (RQ1b); WiReady field inference (RQ5); C4 parts 3–4 (RQ2, RQ3); C1 as target schema |
| (2) Robust pipelines for automated analysis of existing datasets by AI tools and systems | C2 (RQ4); C3 (RQ5, RQ6) |
| (3) Augment and/or harmonize existing datasets | C4 (RQ7, RQ8), built on C1–C3 |

### References Cited

*(Submitted as the separate References Cited document; the Project Description keeps only the bracketed markers.)*

[1] Joint Committee for Guides in Metrology. *JCGM 200:2012, International Vocabulary of Metrology: Basic and General Concepts and Associated Terms (VIM)*, 3rd ed., entry 2.27 "definitional uncertainty." BIPM, 2012. https://www.bipm.org/en/committees/jc/jcgm/publications

[2] U.S. National Science Foundation. *AI Datasets: Unlocking Dataset Value for AI-Enabled Scientific Discovery*. Program Solicitation NSF 26-512, 2026. https://www.nsf.gov/funding/opportunities/ai-datasets-unlocking-dataset-value-ai-enabled-scientific-discovery/nsf26-512/solicitation

[3] 3rd Generation Partnership Project. *TS 38.215 V18.4.0, NR; Physical layer measurements (Release 18)*; published as ETSI TS 138 215 V18.4.0 (2025-01). https://www.etsi.org/deliver/etsi_ts/138200_138299/138215/18.04.00_60/ts_138215v180400p.pdf

[4] MLCommons Croissant Working Group. *Croissant Format Specification, Version 1.1*. January 29, 2026. https://docs.mlcommons.org/croissant/docs/croissant-spec-1.1.html

[5] T. Lebo, S. Sahoo, D. McGuinness (eds.). *PROV-O: The PROV Ontology*. W3C Recommendation, April 30, 2013. https://www.w3.org/TR/prov-o/

[6] SigMF Project. *SigMF: The Signal Metadata Format*. Specification and reference implementation. https://github.com/sigmf/SigMF

[7] M. Bagnulo, B. Claise, P. Eardley, A. Morton, A. Akhter. *Registry for Performance Metrics*. IETF RFC 8911, November 2021. https://www.rfc-editor.org/rfc/rfc8911.html

[8] ETSI. *TS 102 250-6 V1.3.1 (2019-11), Speech and multimedia Transmission Quality (STQ); QoS aspects for popular services in mobile networks; Part 6: Post processing and statistical methods*. https://www.etsi.org/deliver/etsi_ts/102200_102299/10225006/01.03.01_60/ts_10225006v010301p.pdf

[9] 3GPP / ETSI. *TS 32.404 V19.0.0 (Release 19), Performance Management (PM); Performance measurements; Definitions and template*; published as ETSI TS 132 404 V19.0.0 (2025-10). https://www.etsi.org/deliver/etsi_ts/132400_132499/132404/19.00.00_60/ts_132404v190000p.pdf

[10] V. Paxson. "Strategies for Sound Internet Measurement." In *Proc. ACM Internet Measurement Conference (IMC)*, 2004, pp. 263–271. https://doi.org/10.1145/1028788.1028824

[11] A. H. Fahim Raouf, D. Lee, M. Rahman, S. Masrur, G. Reddy, C. Dickerson, M. S. Hossen, S. Vargas Villar, A. Gürses, S. Singh, S. J. Maeng, M. Ezuma, C. Roberts, M. R. Sarbudeen, T. J. Zajkowski, M. Mushi, O. Ozdemir, R. Asokan, I. Guvenc, M. L. Sichitiu, R. Dutta. "Curated Wireless Datasets for Aerial Network Research." arXiv:2510.08752v2, March 2026 (v1 October 2025). https://arxiv.org/abs/2510.08752

[12] B. Kaplan, S. Keşir, A. F. Coşkun. "A Comparative and Measurement-Based Study on Real-Time Network KPI Extraction Methods for 5G and Beyond Applications." arXiv:2504.16039, April 2025. https://arxiv.org/abs/2504.16039

[13] National Institute of Standards and Technology, Information Technology Laboratory. *Statistical Reference Datasets (StRD)*. https://www.itl.nist.gov/div898/strd/

[14] D. Xie, Y. Li, M. Kim, H. V. Pham, L. Tan, X. Zhang, M. W. Godfrey. "DocTer: Documentation-Guided Fuzzing for Testing Deep Learning API Functions." In *Proc. ACM ISSTA*, 2022. https://arxiv.org/abs/2109.01002

[15] J. C. Brønd, L. B. Andersen, D. Arvidsson. "Generating ActiGraph Counts from Raw Acceleration Recorded by an Alternative Monitor." *Medicine & Science in Sports & Exercise* 49(11):2351–2360, 2017. https://doi.org/10.1249/MSS.0000000000001344

[16] J. C. Knight, N. G. Leveson. "An Experimental Evaluation of the Assumption of Independence in Multiversion Programming." *IEEE Transactions on Software Engineering* SE-12(1):96–109, 1986. https://doi.org/10.1109/TSE.1986.6312924

[17] SLSA. *Verification Summary Attestation (VSA)*, SLSA Specification v1.0. https://slsa.dev/spec/v1.0/verification_summary

[18] R. Albertoni, A. Isaac (eds.). *Data on the Web Best Practices: Data Quality Vocabulary*. W3C Working Group Note, December 15, 2016. https://www.w3.org/TR/vocab-dqv/

[19] K. P. Mok (PI), K. C. Claffy (co-PI). "CICI: IPAAI: CANIS: Curated AI-ready Network telescope datasets for Internet Security." NSF Award 2531134, University of California San Diego, 2025–2028. https://www.nsf.gov/awardsearch/show-award/?AWD_ID=2531134

[20] C. Cao (PI), T. K. Samuel, G. Rogers (co-PIs). "CICI: IPAAI: CERTIFY: Contractual Integrity for Scientific AI-Ready Data Pipelines." NSF Award 2613350, University of Tennessee Knoxville, 2026–2029. https://www.nsf.gov/awardsearch/show-award/?AWD_ID=2613350

[21] FG Security in Telecommunications, TU Berlin. *SCAT: Signaling Collection and Analysis Tool* (Qualcomm and Samsung baseband diagnostic-log parser). Open-source software. https://github.com/fgsect/scat

[22] Y. Li, C. Peng, Z. Yuan, J. Li, H. Deng, T. Wang. "MobileInsight: Extracting and Analyzing Cellular Network Information on Smartphones." In *Proc. ACM MobiCom*, 2016. https://doi.org/10.1145/2973750.2973751 (KPI analyzers: https://github.com/mobile-insight/mobileinsight-core)

[23] P1 Security. *QCSuper* (Qualcomm diagnostic-log capture and decoding tool). Open-source software. https://github.com/P1sec/QCSuper

[24] K. Hiniduma, S. Byna, J. L. Bez, R. Madduri. "AI Data Readiness Inspector (AIDRIN) for Quantitative Assessment of Data Readiness for AI." In *Proc. SSDBM*, 2024. arXiv:2406.19256. https://arxiv.org/abs/2406.19256

[25] A. Devaraju, R. Huber. "An Automated Solution for Measuring the Progress toward FAIR Research Data." *Patterns* 2(11):100370, 2021. https://doi.org/10.1016/j.patter.2021.100370

[26] S. R. Wilkinson, P. Shpilker, W. Brewer. "SetGo: Metadata Readiness for Scientific AI Datasets." In *Proc. SSDBM*, 2026. arXiv:2607.22677. https://arxiv.org/abs/2607.22677

[27] H. Sun, W. W. Cohen, R. Salakhutdinov. "ConditionalQA: A Complex Reading Comprehension Dataset with Conditional Answers." In *Proc. ACL*, 2022, pp. 3627–3637. https://doi.org/10.18653/v1/2022.acl-long.253

[28] B. P. Majumder, H. Surana, D. Agarwal, B. Dalvi Mishra, A. Meena, A. Prakhar, T. Vora, T. Khot, A. Sabharwal, P. Clark. "DiscoveryBench: Towards Data-Driven Discovery with Large Language Models." arXiv:2407.01725, 2024. https://arxiv.org/abs/2407.01725

[29] Q. Sun, Z. Liu, C. Ma, Z. Ding, F. Xu, Z. Yin, H. Zhao, Z. Wu, K. Cheng, Z. Liu, J. Wang, Q. Li, X. Tang, T. Xie, X. Feng, X. Li, B. Kao, W. Wang, B. Qi, L. Kong, Z. Wu. "ScienceBoard: Evaluating Multimodal Autonomous Agents in Realistic Scientific Workflows." In *Proc. ICLR*, 2026. https://arxiv.org/abs/2505.19897

[30] A. Narayanan, E. Ramadan, R. Mehta, X. Hu, Q. Liu, R. A. K. Fezeu, U. K. Dayalan, S. Verma, P. Ji, T. Li, F. Qian, Z.-L. Zhang. "Lumos5G: Mapping and Predicting Commercial mmWave 5G Throughput." In *Proc. ACM IMC*, 2020, pp. 176–193. https://doi.org/10.1145/3419394.3423629

[31] A. Narayanan, E. Ramadan, J. Carpenter, Q. Liu, Y. Liu, F. Qian, Z.-L. Zhang. "A First Look at Commercial 5G Performance on Smartphones." In *Proc. The Web Conference (WWW)*, 2020, pp. 894–905. https://doi.org/10.1145/3366423.3380169

[32] A. Hassan, A. Narayanan, A. Zhang, W. Ye, R. Zhu, S. Jin, J. Carpenter, Z. M. Mao, F. Qian, Z.-L. Zhang. "Vivisecting Mobility Management in 5G Cellular Networks." In *Proc. ACM SIGCOMM*, 2022, pp. 86–100. https://doi.org/10.1145/3544216.3544217

[33] R. A. K. Fezeu, C. Fiandrino, E. Ramadan, J. Carpenter, L. Coelho de Freitas, F. Bilal, W. Ye, J. Widmer, F. Qian, Z.-L. Zhang. "Unveiling the 5G Mid-Band Landscape: From Network Deployment to Performance and Application QoE." In *Proc. ACM SIGCOMM*, 2024, pp. 358–372. https://doi.org/10.1145/3651890.3672269

[34] M. I. Rochman, W. Ye, Z.-L. Zhang, M. Ghosh. "A Comprehensive Real-World Evaluation of 5G Improvements Over 4G in Low- and Mid-Bands." *IEEE Transactions on Cognitive Communications and Networking* 11(3):1427–1441, June 2025. https://doi.org/10.1109/TCCN.2025.3558062 ; dataset: https://github.com/ndghoshlab/mpls-chi-2022-2024

[35] Infovista. *TEMS Discovery* (multi-vendor drive-test log post-processing), product documentation. https://www.infovista.com/products/tems-discovery/network-testing-analytics

[36] K. MacMillan, T. Mangla, J. Saxon, N. P. Marwell, N. Feamster. "A Comparative Analysis of Ookla Speedtest and Measurement Labs Network Diagnostic Test (NDT7)." *Proc. ACM on Measurement and Analysis of Computing Systems (POMACS)* 7(1), 2023. https://doi.org/10.1145/3579448

[37] V. Paxson. "On Calibrating Measurements of Packet Transit Times." In *Proc. ACM SIGMETRICS*, 1998. https://doi.org/10.1145/277851.277865

[38] S. B. Moon, P. Skelly, D. Towsley. "Estimation and Removal of Clock Skew from Network Delay Measurements." In *Proc. IEEE INFOCOM*, 1999. https://doi.org/10.1109/INFCOM.1999.749287

[39] Y. C. Zhang, S. Zhang, M. Liu, E. Daly, S. Battalio, S. Kumar, B. Spring, J. M. Rehg, N. Alshurafa. "SyncWISE: Window Induced Shift Estimation for Synchronization of Video and Accelerometry from Wearable Sensors." *Proc. ACM IMWUT* 4(3), 2020. https://doi.org/10.1145/3411824

[40] S. Steegen, F. Tuerlinckx, A. Gelman, W. Vanpaemel. "Increasing Transparency Through a Multiverse Analysis." *Perspectives on Psychological Science* 11(5):702–712, 2016. https://doi.org/10.1177/1745691616658637

[41] C. G. Northcutt, A. Athalye, J. Mueller. "Pervasive Label Errors in Test Sets Destabilize Machine Learning Benchmarks." In *NeurIPS Datasets and Benchmarks Track*, 2021. https://arxiv.org/abs/2103.14749

[42] E. Kim, A. Garg, K. Peng, N. Garg. "Correlated Errors in Large Language Models." In *Proc. ICML*, 2025. https://arxiv.org/abs/2506.07962

[43] J. Huang, X. Chen, S. Mishra, H. S. Zheng, A. W. Yu, X. Song, D. Zhou. "Large Language Models Cannot Self-Correct Reasoning Yet." In *Proc. ICLR*, 2024. https://arxiv.org/abs/2310.01798

[44] C. Qu, S. Dai, X. Wei, H. Cai, S. Wang, D. Yin, J. Xu, J.-R. Wen. "From Exploration to Mastery: Enabling LLMs to Master Tools via Self-Driven Interactions" (DRAFT). In *Proc. ICLR*, 2025. https://arxiv.org/abs/2410.08197

[45] W. Fang, Y. Zhang, K. Qian, J. Glass, Y. Zhu. "PLAY2PROMPT: Zero-shot Tool Instruction Optimization for LLM Agents via Tool Play." In *Findings of ACL*, 2025. https://arxiv.org/abs/2503.14432

[46] Y. Liu, S. Hou, X. Chen, Y. Wang, S. Yuan, Q. Gan, S. You, F. Chen, W. Li, S. Hu, Y. Liu, E. Xia, X. Lu, H. Wang, F. Xu, Y. Li. "ZhuLong: Execution-Grounded LLM Agent for EDA Scripting with Offline API Self-Exploration." arXiv:2608.07925, 2026. https://arxiv.org/abs/2608.07925

[47] F. Yi, H. Wan, K. Jamieson, O. Michel. "Automated, Cross-Layer Root Cause Analysis of 5G Video-Conferencing Quality Degradation" (Domino). arXiv:2505.14540, May 2025. https://arxiv.org/abs/2505.14540

[48] A. Alkhateeb, G. Charan, T. Osman, A. Hredzak, J. Morais, U. Demirhan, N. Srinivas. "DeepSense 6G: A Large-Scale Real-World Multi-Modal Sensing and Communication Dataset." *IEEE Communications Magazine* 61(9):122–128, September 2023. https://doi.org/10.1109/MCOM.006.2200730

[49] M. Sana, N. Piovesan, A. De Domenico, Y. Kang, H. Zhang, M. Debbah, F. Ayed. "Reasoning Language Models for Root Cause Analysis in 5G Wireless Networks." arXiv:2507.21974, July 2025. https://arxiv.org/abs/2507.21974 ; dataset: https://huggingface.co/datasets/netop/TeleLogs

[50] A. Feng, A. Varvarigos, I. Panitsas, D. Fernandez, J. Wei, Y. Guo, J. Chen, A. Maatouk, L. Tassiulas, R. Ying. "TelecomTS: A Multi-Modal Observability Dataset for Time Series and Language Analysis." In *Proc. ICML*, 2026. https://arxiv.org/abs/2510.06063 ; data: https://github.com/Ali-maatouk/TelecomTS



---

## Internal notes for the PI (do not submit)

- LENGTH: The prose (every non-table line of the body, headings included) runs 1,543 words. The two tables add about 225 more (1,829 words by wc for the whole body, table syntax included; references excluded). The RQ table is new and was added at the review's request. If you want the prose under 1,500, the cheapest cuts are the SINR passport example in C1 (20 words) and the 3GPP clause (10 words). If the RQ table moves to the Research Plan, add one line here that points to it.
- PLACEHOLDERS IN THE BODY. C4: [X TB] of raw logs, cited [N] times, hosted at [repository]. Method: the [X]% Year-1 gate. RQ table: RQ1 [Year 1: X]; RQ1b [≥X%]; RQ3 [X%]; RQ4 [X%]; RQ5 [≥X% precision]; RQ6 [≥Y%]; RQ8 [2–3] findings. For RQ8, confirm the example findings: throughput prediction in [30] and handover behavior in [32]. The C2 plan also needs a number: how many 'codes behind headline KPIs' are in scope for Year 1.
- C4 FACTS TO CONFIRM. (a) The per-campaign tool split comes from the corpus slide: [33, 34] used XCAL, [32] used chipset-level logs (did it use XCAL?), [30, 31] used phone-side app logs. Please also give tool and version, phones, and raw-log volume for each campaign. (b) I wrote 'one joint with the Ghosh lab' for [34]. The first author is Rochman and the last is Ghosh, the UMN co-authors are Ye and Zhang, and the data are hosted under ndghoshlab. Confirm the wording, and confirm that the group holds that campaign's raw logs. (c) The partner step is now conditional ('where letters of collaboration exist'), and no track is implied.
- CAMERA AND LIDAR: I removed camera from the body, as the review asked (the inventory is unconfirmed). If by Oct 14 the inventory confirms time and place overlap with the .drm sessions, restore it as one clause in C4 part (3). LiDAR stays out.
- PLAN STATEMENT TO VERIFY: C2 says the independent path will 'pair our container reader with open payload decoders [21–23]'. That only works if the payloads inside .drm records are standard Qualcomm (or Samsung) DIAG log packets that SCAT, MobileInsight or QCSuper can parse. If they are not, change it to 'extend our reader's payload decoding'.
- INDEPENDENCE STATUS: Vault B1 records the reader matching XCAP's tables with 0.00% error on 3 traces x 2 codes, but the deck says 'no independent value check yet'. The body reconciles the two: matches exist, but none has passed the independence test, so no value is certified. Please confirm whether the reader's decoding of those two codes was worked out by looking at XCAP's tables. If it was, it fails the independence test by construction. If it was not, those values could become the first certified candidates.
- LEGAL: If the reader cannot be released, the body commits to releasing its test outputs plus the check library. I did not adopt the reviewer's 'hosted verification service', because running the reader as a service carries the same legal exposure as releasing it. The university check is still High risk.
- NUMBERS CUT FROM THE BODY (safe to restore in a Preliminary Results table): 15.3 GB parsed at 283 MB/s with zero resyncs; four XCAL collection product lines; version label 5.33.0.15 vs build paths 5.33.0.9; 81,605 streaming rows; 5,028.6 ms phone-clock jitter; the reader's 0.00% error on 2 codes x 3 traces; the 50/50 sampling detail (18 parameters x 4 statistics, fixed seed, ping trace only); the measured-Sum field that conflicts with the rule table, which has no SUM entry (B4: 840 vs 3,481 rows, equal totals). About 5,500 semantically checked field values were also cut. If you restore that one, write 'about 5,500 field values out of 42.87 M records', and first reconcile the 46.98 M vs 42.87 M scopes that insight.json flags.
- NUMBERS STILL EXCLUDED ON PURPOSE: 21.33% vs 17.00% (parameter unconfirmed); 165/166 = 99.4%; 87% of handovers; 17.6 s; 310 handovers; the Confident-Wrong candidate; the P5 percentages; the open-book gain.
- CHECKS I MODIFIED OR REJECTED. (1) The reviewer asked for a clause saying the 7 dB SINR gap matters in practice. I did not use the suggested 'link unusable / handover trigger' wording, because the vault does not support it. I wrote 'a fivefold gap in linear terms' instead, which is arithmetic. (2) I rejected the reviewer's 'none records both the sampling setting and the averaging rule'. It is false for [34], whose README states 1 s averaging and mode. The body says 'full export configuration' instead: ndghoshlab omits the XCAP version, settings and timezone, and Vienna says its export configurations are not included. (3) The reviewer suggested opening each 'New:' with 'the first ...'. That breaks stress_verdict, so each 'New:' now opens with the positive content followed by 'which we found nowhere'. (4) The reviewer asked to cut 'What we do not claim' to two sentences. I shortened it but kept the concessions stress_verdict requires. (5) I did not use the reviewer's 'remote driving' gloss for teleoperation, because the vault does not confirm it. Please say what the teleop workload is, and whether the 602 stalls come from teleop or from streaming. (6) I kept the deck's tier names (certified/provisional/unverified) rather than 'independently reproduced', and removed 'ships'. (7) I did not add 'XCAL produced N of M public datasets', because no such survey exists. A systematic audit of 15-20 releases would supply both that figure and a real denominator for the five-release check. (8) I did not use Check 2's rewrite of the 5,500 figure ('agree between our reader and the tool'). MASTER-STATUS 09-02 ties the 5,500 to semantic checks of the tool's .dat tables, not to agreement between our reader and the tool. (9) The 50/50 rewrites in Check 2 and the review mixed up the excluded mirror (xCOMMON_BMF_*) with the tables that were actually compared. The body now separates them. (10) The References list stays in this output because the task requires it. It is headed 'References Cited', so paste only the bracketed markers into the Project Description.
- DECK UPDATES TO MATCH THIS SECTION: (a) The idea slide says 'Comparable corpora'; this section says 'Comparable existing corpora'. Pick one. (b) rqresults and agent say 'an agent operated the tool from a shell'. Change that to 'solved tasks mostly by reading the tool's model files'. (c) The agent notes describe ZhuLong's admission criterion as 'task success or self-made examples'. Change it to 'its own sandbox runs'. (d) governance marks the Croissant working group '[to contact]', which matches the body's 'to be proposed'. (e) The mapping table puts WiReady's field inference under synopsis item 1 as well as item 2, but T2 lists RQ5 only under item 2. Decide which you want.
- REFERENCES TO SPOT-CHECK: [21] is credited to TU Berlin FG SecT as software because I could not confirm an author list. Confirm the PI initials 'K. P. Mok' in [19]. [3] is V18.4.0, which the quote is pinned to; V18.5.0 (2026-07) also exists. [26] is cited as SSDBM 2026, based on its arXiv comment. [50] uses arXiv 2510.06063. Before Nov 4, check whether Domino [47] has released its datasets.
- SIGCOMM '27 BOUNDARY: The body names no mechanisms and no venue. Keep these notes internal only. If the supervised, blind XCAP loop cycle runs before Nov 4, add one sentence with its result to the Method paragraph. Until then, 'the engine is untested; we ran its loop by hand' stands.
