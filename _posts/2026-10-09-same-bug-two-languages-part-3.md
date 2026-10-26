---
title: Same Bug, Two Languages, Part 3 - The Dictionary
date: 2026-10-24
categories:
  - ai
  - llm
  - owasp
  - security
  - compliance
tags:
  - ai
  - security
  - iso27001
  - nis2
  - en
mermaid: false
published: true
render_with_liquid: "false"
permalink: /:year/:month/:day/:title:output_ext
---

## From the pipeline to the risk register

[Part 1](https://zahlenhelfer.github.io/2026/10/07/same-bug-two-languages-part-1.html) picked the target. [Part 2](https://zahlenhelfer.github.io/2026/10/09/same-bug-two-languages-part-2.html) let one poisoned ticket carry a production key out of a boring ticket system, and ended with a table: what I did in code, and what the [OWASP AI Exchange](https://owaspai.org/) calls it.

That table is for developers. Now I walk it over to the other side of the building.

On the other side nobody reads regular expressions. There is a risk register, a Statement of Applicability, an auditor with a checklist, and since last December in Germany a law with a [management liability clause](https://www.gesetze-im-internet.de/bsig_2025/__38.html). A finding like "the regex missed `TK-DASH-ZX44-Qh9`" does not survive that trip. It gets lost somewhere between the Jira ticket and the quarterly risk review.

So this part translates the attack, line by line. And I do not invent the threat model for it. Somebody already did that.

## We don't invent the threat model

The OWASP AI Exchange is a vendor-neutral threat model for AI systems. Every threat has a number, every control has an ID a developer can act on: [#INPUT SEGREGATION](https://owaspai.org/go/inputsegregation), [#SENSITIVE OUTPUT HANDLING](https://owaspai.org/go/sensitiveoutputhandling), [#LEAST MODEL PRIVILEGE](https://owaspai.org/go/leastmodelprivilege). Three parts of it do most of the translation work.

**Threat to control.** The [periodic table of AI security](https://owaspai.org/go/periodictable) lists the controls per threat. For direct prompt injection: limit unwanted behaviour, monitor, rate limit, model access control, prompt injection I/O handling, model alignment. For indirect prompt injection: limit unwanted behaviour, prompt injection I/O handling, input segregation. For disclosure in output: sensitive data limitation, monitor, rate limit, model access control, sensitive output handling. The threat sections add the point that matters most: _**prompt injection is hard to prevent, so blast radius control is key.**_

**Control to other frameworks.** Every control links to [OpenCRE](https://owaspai.org/docs/ai_security_overview/), which "links all AI security controls and threats across standards and guidelines". For our controls that means NIST AI 100-2, MITRE ATLAS, ENISA and ETSI.

**Control to ISO.** Many controls carry a "Useful standards" block. If ISO 27002 has a matching control, the Exchange names it and rates the gap. If it has none, it says so. Keep that second sentence in mind. It decides the second half of this post. (The Exchange cites ISO 27002 controls. Their numbers and titles are identical to ISO 27001 Annex A, so they go straight into a Statement of Applicability. 27002 adds the implementation guidance, 27001 is what gets audited.)

### Why your auditor should care where it came from

The Exchange is not a standard. But its content went into two of them.

**ISO/IEC 27090** was [published in October 2026](https://www.iso.org/standard/27090): guidance on threats to AI systems, prompt injection among them, according to the [summary on iso27001security.com](https://www.iso27001security.com/html/27090). OWASP [contributed 70 pages](https://owasp.github.io/blog/2025/05/06/AI-Exchage-Regulation), and the Exchange founder was part of the working group.

**prEN 18282** is the draft harmonised standard for the cybersecurity part of the AI Act, [Article 15(5)](https://standards.iteh.ai/catalog/standards/sist/6e96fa2e-6767-4acd-a48b-379a110bcf19/osist-pren-18282-2026), for high-risk AI systems. The [AI Exchange overview](https://owaspai.org/docs/ai_security_overview/) counts 70 contributed pages here as well, and the Exchange founder is co-editor. The enquiry draft defines direct and indirect prompt injection as terms of their own (3.2.15, 3.2.20).

Neither is something you get certified against. 27090 is guidance. 18282 will give presumption of conformity for high-risk AI once it is published and cited in the Official Journal, and our ticket bot is most likely not high-risk. So why bother?

Because of one sentence in [§ 30 Abs. 2 BSIG](https://www.gesetze-im-internet.de/bsig_2025/__30.html), the German implementation of [NIS-2 Article 21](https://www.nis-2-directive.com/NIS_2_Directive_Article_21.html): the measures shall follow the state of the art ("Stand der Technik") and take the relevant European and international standards into account. That is what your auditor measures you against. And the state of the art for AI security is now written down, in two standards that partly came from the same document your developers already use.

So when your engineer cites an Exchange ID, they cite the text that now defines that state of the art.

## The findings

Part 2 in auditor-friendly form. Same tests, new IDs.

| ID | Finding | Evidence from part 2 |
| --- | --- | --- |
| F-01 | Credential in the model's context | Recovery key sits in the system prompt. Found by reading the code |
| F-02 | Direct prompt injection resisted by the system prompt alone | Four refusals. One sentence made the difference |
| F-03 | Indirect prompt injection via external ticket | Key printed at the end of every summary |
| F-04 | Input segregation not effective | Tags plus rules: key printed anyway |
| F-05 | Output filter bypassed by reformatting | Regex missed `TK-DASH-ZX44-Qh9`, UI reported success |
| F-06 | Non-deterministic test results | One bypass in eight runs |
| F-07 | Secret disclosed in production | Not a test finding. What happens when F-01 to F-05 line up for real |

F-07 is the odd one out on purpose. Tests find weaknesses, incidents are found by monitoring. A dictionary without that row ends exactly where the auditor starts asking.

## The dictionary

Two halves, same IDs, so you can read across.

### Half one: what the Exchange says

| ID | AI Exchange threat | AI Exchange controls | ISO/IEC 27090 | prEN 18282 (draft) |
| --- | --- | --- | --- | --- |
| F-01 | [2.3.1 Disclosure of sensitive data in model output](https://owaspai.org/go/disclosureinoutput) | [Sensitive data limitation](https://owaspai.org/go/datalimit): [#DATA MINIMIZE](https://owaspai.org/go/dataminimize); plus #LEAST MODEL PRIVILEGE | Clause 7 | 10.4 |
| F-02 | [2.2.1 Direct prompt injection](https://owaspai.org/go/directpromptinjection) | [#MODEL ALIGNMENT](https://owaspai.org/go/modelalignment), [#PROMPT INJECTION I/O HANDLING](https://owaspai.org/go/promptinjectioniohandling), [#MODEL ACCESS CONTROL](https://owaspai.org/go/modelaccesscontrol), [#RATE LIMIT](https://owaspai.org/go/ratelimit), [#MONITOR USE](https://owaspai.org/go/monitoruse) | Clauses 6, 7 | 3.2.15, 10.6 |
| F-03 | [2.2.2 Indirect prompt injection](https://owaspai.org/go/indirectpromptinjection) | #PROMPT INJECTION I/O HANDLING, #INPUT SEGREGATION, #LEAST MODEL PRIVILEGE, [#OVERSIGHT](https://owaspai.org/go/oversight) | Clauses 6, 7 | 3.2.20, 10.6 |
| F-04 | 2.2.2 Indirect prompt injection | #INPUT SEGREGATION, its limitations | Clause 7 | 11 |
| F-05 | 2.3.1 Disclosure of sensitive data in model output | #SENSITIVE OUTPUT HANDLING, #PROMPT INJECTION I/O HANDLING | Clause 7 | 10.4 |
| F-06 | 2.2 Prompt injection | [5.3.4 Testing against prompt injection](https://owaspai.org/go/testingpromptinjection) | Clause 7 | 11, 12 |
| F-07 | 2.3.1 Disclosure of sensitive data in model output | #MONITOR USE, [#SEC PROGRAM](https://owaspai.org/go/secprogram) (incident response) | Clause 7 | 10.4 |

Two cells are mine, not the Exchange's. For F-01 the periodic table lists only sensitive data limitation. #LEAST MODEL PRIVILEGE is my addition, for the second way a key gets in: an assistant that may read every runbook. And for F-03 the Exchange also lists #MODEL ACCESS CONTROL and #RATE LIMIT. I left them out because the attacker is not a chat user. They just send an e-mail.

The 27090 and 18282 columns are on clause level. 27090 is paywalled and days old, and 18282 is a draft whose numbers will move. Do not put them into a contract.

### Half two: what the auditor reads

Here it gets honest. The second column holds what the Exchange itself names in its "Useful standards" blocks. The third holds what I added. If you only read the second column, you see how far "we don't invent the mapping" really carries.

| ID | ISO 27001 Annex A, named by the Exchange | ISO 27001 Annex A, my addition | NIS-2 (Directive 2022/2555) | BSIG (German implementation) | Owner |
| --- | --- | --- | --- | --- | --- |
| F-01 | 8.2 (#LEAST MODEL PRIVILEGE). #DATA MINIMIZE: "not covered yet" | 5.12 Classification, 5.17 Authentication information | Art. 21(2)(i) access control, asset management; (j) MFA | § 30 Abs. 2 Nr. 9, 10 | Platform team |
| F-02 | 5.15, 5.16, 5.18, 5.3, 8.3 (#MODEL ACCESS CONTROL); 8.15, 8.16 (#MONITOR USE). #RATE LIMIT: "no control for this". #MODEL ALIGNMENT, #PROMPT INJECTION I/O HANDLING: none | – | Art. 21(2)(i) access control; (b) incident handling | § 30 Abs. 2 Nr. 9, 2 | App team |
| F-03 | 8.2 (#LEAST MODEL PRIVILEGE). #INPUT SEGREGATION, #PROMPT INJECTION I/O HANDLING: none | 8.25 Secure development life cycle, 8.26 Application security requirements, 8.28 Secure coding | Art. 21(2)(e) development and maintenance | § 30 Abs. 2 Nr. 5 | App team |
| F-04 | #INPUT SEGREGATION: none | Clause 9.1 Monitoring and evaluation, 8.29 Security testing | Art. 21(2)(f) effectiveness of measures | § 30 Abs. 2 Nr. 6 | ISMS |
| F-05 | #SENSITIVE OUTPUT HANDLING: "not covered yet" | 8.11 Data masking, 8.12 Data leakage prevention | Art. 21(2)(e) development; (f) effectiveness | § 30 Abs. 2 Nr. 5, 6 | App team |
| F-06 | Testing section: none | 8.25 Secure development life cycle, 8.29 Security testing | Art. 21(2)(f) effectiveness of measures | § 30 Abs. 2 Nr. 6 | App team + ISMS |
| F-07 | 8.15, 8.16 (#MONITOR USE); 27001 to 27005 (#SEC PROGRAM); NIS-2 24h/72h/1 month (#CHECK COMPLIANCE) | 5.24 to 5.26 Incident management, 5.28 Collection of evidence | Art. 21(2)(b) incident handling; Art. 23 reporting | § 30 Abs. 2 Nr. 2; [§ 32](https://www.gesetze-im-internet.de/bsig_2025/__32.html) | Incident manager |

The BSIG column is not a second opinion. § 30 Abs. 2 BSIG takes over the ten areas of Article 21(2) in the same order, so letter (a) is Nr. 1 and letter (j) is Nr. 10. And Article 23 lives in § 32. Use the column that matches your auditor.

The NIS-2 and BSIG columns are mine as well, with one exception: the regulatory map in [#CHECK COMPLIANCE](https://owaspai.org/go/checkcompliance) sends serious incidents and the NIS-2 timelines to incident response under #SEC PROGRAM plus #MONITOR USE. That is F-07, word for word.

Follow F-03 through both tables. In the first, the developer finds "2.2.2 indirect prompt injection" and #PROMPT INJECTION I/O HANDLING. In the second, the auditor finds 8.26 and NIS-2 Article 21(2)(e). Same bug. Two languages. One ID. But look at who wrote what: 8.26 is my translation. In the Exchange's own column, next to the two controls that matter for this attack, it says: none.

## Where the Exchange stops

Look at the second column again. The Exchange maps cleanly to ISO wherever ISO already had the idea: access control, monitoring, privileges. It stops at exactly the controls this attack was about.

**No vocabulary for the AI-specific controls.** #INPUT SEGREGATION, #PROMPT INJECTION I/O HANDLING and #MODEL ALIGNMENT have no ISO 27002 counterpart. #SENSITIVE OUTPUT HANDLING and #DATA MINIMIZE are "not covered yet". F-03, F-04 and F-05 all land there. Take F-05: 8.12 data leakage prevention becomes the hook in the register, and #SENSITIVE OUTPUT HANDLING becomes the actual content. A classic DLP thinks in files, mails and USB sticks. It has never seen a model spell out a key with DASH because a ticket asked politely. Write "8.12, implemented", and the auditor ticks it off while the key walks out.

**F-04 is a finding about a control, not about the system.** Input segregation was in place, documented, and did not work. Annex A has no AI-specific home for that, so it lands in clause 9.1 and NIS-2 letter (f), the effectiveness of measures. I think (f) is the most underrated letter in Article 21. Everybody implements measures. Few measure whether they work.

**Non-determinism breaks the pass/fail habit.** The Exchange asks to [run the test multiple times](https://owaspai.org/go/testingpromptinjection). Audit evidence loves a green check mark. Honest evidence is a rate: attempts, successes, model, version, date. And an attacker sends fifty tickets for free, so the likelihood is not "1 in 8". It is "eventually".

**Only one row removes the risk.** F-02 to F-06 all reduce likelihood. Only F-01, the key in the vault or replaced by SSO, takes the prize away. In ISO 27001 terms that is the difference between modifying a risk and avoiding it. Make the register say so.

The gaps are not a weakness of the Exchange. They are the honest part of it. Where the old language has no word yet, your organisation has to write the control text itself. The Exchange tells you exactly where.

## How the dictionary survives the trip

The mapping is only useful if it travels with the finding. So do not translate findings in a meeting. Put the dictionary next to the code and the tests, as a versioned file, and let every test result carry its finding ID. The developer in the pipeline log and the risk owner in the register then look up the same row.

Two rules make that work. A finding without an entry in the dictionary is an error, not a footnote. And a finding never leaves the register just because the test is green now. F-05 is a good example: the filter in the repository is already stricter than the regex shown in part 2. It normalises the answer first and catches the DASH variant. In the register, F-05 does not disappear. It changes to "treated", with the regression test as evidence.

## Same bug, two languages, one dictionary

Back to the forest from part 1. The developer saw the big cat in part 2. The auditor has not seen it yet, because the auditor looks at a different picture: the risk register. The dictionary paints the cat into that picture too. Same spot, same cat, other language. And where the other language has no word for the cat yet, the dictionary says so.

If you take only one row with you: _**the only one that removes the risk is F-01. Everything else is a better guess. Put the key in the vault.**_

## Sources

- [llm-owasp-ticketsystem](https://github.com/zahlenhelfer/llm-owasp-ticketsystem), the code for this series
- OWASP AI Exchange:
  - [AI security overview](https://owaspai.org/docs/ai_security_overview/) (contributions to ISO/IEC 27090 and prEN 18282, OpenCRE), [Periodic table of AI security](https://owaspai.org/go/periodictable)
  - [2.2.1 Direct prompt injection](https://owaspai.org/go/directpromptinjection), [2.2.2 Indirect prompt injection](https://owaspai.org/go/indirectpromptinjection), [2.3.1 Disclosure of sensitive data in model output](https://owaspai.org/go/disclosureinoutput)
  - [#CHECK COMPLIANCE](https://owaspai.org/go/checkcompliance) (regulatory map, status of ISO/IEC 27090 and prEN 18282), [#SEC PROGRAM](https://owaspai.org/go/secprogram)
  - [1.2 Sensitive data limitation](https://owaspai.org/go/datalimit), [#DATA MINIMIZE](https://owaspai.org/go/dataminimize), [#LEAST MODEL PRIVILEGE](https://owaspai.org/go/leastmodelprivilege), [#OVERSIGHT](https://owaspai.org/go/oversight), [#MODEL ALIGNMENT](https://owaspai.org/go/modelalignment)
  - [#MONITOR USE](https://owaspai.org/go/monitoruse), [#RATE LIMIT](https://owaspai.org/go/ratelimit), [#MODEL ACCESS CONTROL](https://owaspai.org/go/modelaccesscontrol)
  - [#PROMPT INJECTION I/O HANDLING](https://owaspai.org/go/promptinjectioniohandling), [#INPUT SEGREGATION](https://owaspai.org/go/inputsegregation), [#SENSITIVE OUTPUT HANDLING](https://owaspai.org/go/sensitiveoutputhandling)
  - [5.3.4 Testing against prompt injection](https://owaspai.org/go/testingpromptinjection)
- [OWASP Enables AI Regulation That Works with OWASP AI Exchange](https://owasp.github.io/blog/2025/05/06/AI-Exchage-Regulation), OWASP blog, 6 May 2025
- [ISO/IEC 27090:2026](https://www.iso.org/standard/27090), and the [ISO/IEC 27090 summary](https://www.iso27001security.com/html/27090) on iso27001security.com
- [oSIST prEN 18282:2026](https://standards.iteh.ai/catalog/standards/sist/6e96fa2e-6767-4acd-a48b-379a110bcf19/osist-pren-18282-2026), public enquiry draft
- [ISO/IEC 27001:2022](https://www.iso.org/standard/27001); Annex A control titles cross-checked with the [Open Security Architecture listing](https://opensecurityarchitecture.org/frameworks/iso-27001-2022)
- [Directive (EU) 2022/2555 (NIS-2)](https://eur-lex.europa.eu/eli/dir/2022/2555/oj/eng), Articles 21 and 23; [Article 21 text](https://www.nis-2-directive.com/NIS_2_Directive_Article_21.html)
- Mayer Brown, [Cyber rules for essential and important entities take effect in Germany](https://www.mayerbrown.com/en/insights/publications/2025/12/cyber-rules-for-essential-and-important-entities-take-effect-in-germany-nis2-implementing-law), 15 December 2025 (in force since 6 December 2025)
- BSIG: [§ 30 Risikomanagementmaßnahmen](https://www.gesetze-im-internet.de/bsig_2025/__30.html), [§ 32 Meldepflichten](https://www.gesetze-im-internet.de/bsig_2025/__32.html), [§ 38 Pflichten der Geschäftsleitung](https://www.gesetze-im-internet.de/bsig_2025/__38.html)
