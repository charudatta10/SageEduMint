# SageEduMint

📜 SageEduMint University

A P2P‑Credentialed, AI‑Orchestrated, Local‑First Learning Ecosystem

> Version 1.1 — adds content generation via local agents + web scrapers and adaptive exercise generation to the end‑to‑end workflow.




---

Abstract

SageEduMint University is a decentralized, student‑led learning ecosystem that fuses peer‑to‑peer credentialing, AI‑orchestrated adaptive syllabi, and local‑first autonomy. It replaces top‑down academic models with self‑directed, modular education, where learners generate verifiable credentials (VCs) from their daily progress and aggregate them into a portable certificate. The system runs entirely on learner machines, integrating Ollama endpoints, local agents, and peer validators for trust‑minimized credentialing. New in v1.1: a content generation pipeline that uses local agents and configurable web scrapers to curate/source materials, and an adaptive exercise generator that tunes difficulty and modality from prior evaluations.


---

1. Problem Statement

Traditional education suffers from:

Centralized gatekeeping of credentials and opaque assessment.

Static curricula that ignore real‑time learner progress.

Cloud‑locked platforms with weak portability and privacy.

Limited verifiability of learning provenance and outcomes.


Learners demand modular, verifiable, portable learning — peer‑recognized, AI‑adaptive, local‑first, and open/remixable.


---

2. Solution Overview

SageEduMint University proposes:

Markdown‑First Content: Syllabi, content packets, submissions, evaluations, and certificates are human‑readable and Git‑friendly.

Local Agents + Scrapers: YAML‑defined scrapers and tools gather/publicly curate sources; a content distiller composes daily packets with citations.

Adaptive Exercise Generation: Exercises are created per learner, tuned by recent scores and feedback.

Verifiable Credentials: Daily LearningTokens are signed P2P and aggregated into a final certificate.

AI Orchestration (Local): Ollama endpoints generate syllabi, content packets, exercises, and feedback.

P2P Credentialing: Certificates and tokens are portable and independently verifiable.



---

3. System Architecture

3.1 Repository Layout

📁 sage-edumint-university/
├── README.md
├── config/
│   ├── prompts/
│   │   ├── syllabus.md
│   │   ├── daily-content.md
│   │   ├── exercise.md
│   │   ├── evaluation.md
│   │   └── certificate.md
│   └── agents.yaml           # scrapers, validators, tools, RAG settings
├── students/
│   └── <name>/
│       ├── profile.json
│       ├── syllabus.md
│       ├── content/
│       │   └── dayX.md       # curated packet with citations
│       ├── exercises/
│       │   └── dayX.md       # adaptive exercises for the day
│       ├── evaluations/
│       │   └── dayX.md
│       ├── tokens/
│       │   └── dayX.json     # verifiable credential
│       └── certificate.md
├── cli/
│   ├── main.py
│   ├── register.py
│   ├── generate_syllabus.py
│   ├── generate_content.py   # NEW: curated content packets per day
│   ├── generate_exercises.py # NEW: adaptive exercises per day
│   ├── evaluate.py
│   ├── adapt_syllabus.py
│   └── issue_certificate.py
└── templates/
    ├── token-template.json
        ├── certificate-template.md
            └── dashboard.html

            3.2 Core Services

            Identity & Keys: Local DID creation (Ed25519). Keys never leave the host.

            Syllabus Engine: Produces initial roadmap and learning objectives (Markdown).

            Content Engine (NEW): Local agent orchestrates scrapers, parses sources, runs RAG, and emits a daily content packet with citations and attributions.

            Exercise Generator (NEW): Uses learner profile + recent evaluations to generate adaptive exercises (difficulty, modality, scope) mapped to objectives.

            Evaluation Engine: AI/peer evaluation against rubric → feedback + numeric score → signed LearningToken (VC).

            Adaptation Loop: Updates syllabus pacing/topics; triggers regeneration of content and exercises when needed.

            Credentialing: Aggregates tokens → signs certificate (Markdown VC wrapper) referencing token URIs/paths.


            3.3 Data Flow (High‑Level)

            [register] → DID/profile
               ↓
               [generate_syllabus]
                  ↓
                  [generate_content] → dayX.md (citations)
                     ↓
                     [generate_exercises] → dayX.md (adaptive)
                        ↓
                        [student submission]
                           ↓
                           [evaluate] → feedback + score + token(dayX.json)
                              ↓
                              [adapt_syllabus] → (optionally) regen content/exercises
                                 ↓
                                 [issue_certificate] → certificate.md (links all tokens)


                                 ---

                                 4. Agents & Tools

                                 4.1 agents.yaml (Excerpt)

                                 version: 1
                                 ollama:
                                   endpoint: http://localhost:11434
                                     model: mistral:latest
                                     rag:
                                       index: .cache/rag/index
                                         store: sqlite   # or chroma/faiss (local)
                                         scrapers:
                                           - name: markdown_fetcher
                                               type: http
                                                   allow_domains: ["edu", "org", "wikipedia.org", "arxiv.org"]
                                                       deny_patterns: ["paywall", "login"]
                                                           max_pages: 10
                                                             - name: youtube_transcript
                                                                 type: yt_transcript
                                                                     rate_limit_per_min: 10
                                                                     parsers:
                                                                       - name: readability
                                                                           type: html_to_markdown
                                                                           filters:
                                                                             - name: dedupe
                                                                                 type: text_minhash
                                                                                   - name: citecheck
                                                                                       type: citation_enforcer  # ensures sources are listed in outputs
                                                                                       validators:
                                                                                         - name: peer_validator
                                                                                             did: did:example:peer-123
                                                                                               - name: local_agent
                                                                                                   did: did:example:agent-xyz

                                                                                                   4.2 Content Engine

                                                                                                   Discovery: Run configured scrapers; store raw artifacts locally.

                                                                                                   Parsing: Convert HTML/PDF/transcripts → Markdown.

                                                                                                   RAG Build: Index artifacts for semantic retrieval.

                                                                                                   Distillation: Compose a daily packet aligned to syllabus objectives; include source list and inline citations.

                                                                                                   Output: students/<name>/content/dayX.md (Markdown + reference list).


                                                                                                   4.3 Exercise Generator

                                                                                                   Signals: Reads syllabus.md, last N evaluations, and recent content packet.

                                                                                                   Tuning: Adjusts difficulty (Bloom taxonomy), modality (MCQ, coding, proof, essay, project), and scaffolding (hints, exemplars) per learner.

                                                                                                   Alignment: Tags each exercise with objectives and estimated time.

                                                                                                   Output: students/<name>/exercises/dayX.md.



                                                                                                   ---

                                                                                                   5. Workflow (Detailed)

                                                                                                   1. Register

                                                                                                   register.py → DID + profile.json (Ollama endpoint, preferences, goals).



                                                                                                   2. Generate Syllabus

                                                                                                   generate_syllabus.py loads prompts/syllabus.md and student context → writes syllabus.md.



                                                                                                   3. Generate Daily Content (NEW)

                                                                                                   generate_content.py --day X reads syllabus.md to fetch topics for day X.

                                                                                                   Orchestrates scrapers from agents.yaml; builds local RAG; distills content/dayX.md with citations.

                                                                                                   Supports offline mode (uses local cache if network unavailable).



                                                                                                   4. Generate Adaptive Exercises (NEW)

                                                                                                   generate_exercises.py --day X consumes content/dayX.md + recent evaluations/.

                                                                                                   Emits exercises/dayX.md tuned to learner’s pace and gaps.



                                                                                                   5. Student Submission

                                                                                                   Learner writes in exercises/dayX.md (or linked files) and commits.



                                                                                                   6. Evaluation + Token Issuance

                                                                                                   evaluate.py --day X uses prompts/evaluation.md → produces evaluations/dayX.md with rubric scores.

                                                                                                   Signs a LearningToken VC tokens/dayX.json (Ed25519). See §6.



                                                                                                   7. Adaptation Loop

                                                                                                   adapt_syllabus.py adjusts next topics/pace; may trigger regeneration of content/exercises for upcoming days.



                                                                                                   8. Certificate Issuance

                                                                                                   issue_certificate.py verifies all tokens, aggregates metrics, and signs certificate.md with validator DID.





                                                                                                   ---

                                                                                                   6. Verifiable Credentials

                                                                                                   6.1 LearningToken (Daily VC)

                                                                                                   {
                                                                                                     "@context":["https://www.w3.org/2018/credentials/v1"],
                                                                                                       "id":"urn:uuid:{{token_uuid}}",
                                                                                                         "type":["VerifiableCredential","LearningToken"],
                                                                                                           "issuer":"{{agent_did}}",
                                                                                                             "issuanceDate":"{{timestamp}}",
                                                                                                               "credentialSubject":{
                                                                                                                   "id":"{{student_did}}",
                                                                                                                       "day":"{{day_number}}",
                                                                                                                           "exercise":"{{exercise_ref}}",
                                                                                                                               "score":"{{evaluation_score}}",
                                                                                                                                   "feedback":"{{evaluation_summary}}",
                                                                                                                                       "sources": {{source_claims_json}}  
                                                                                                                                         },
                                                                                                                                           "proof":{
                                                                                                                                               "type":"Ed25519Signature2020",
                                                                                                                                                   "created":"{{timestamp}}",
                                                                                                                                                       "proofPurpose":"assertionMethod",
                                                                                                                                                           "verificationMethod":"{{agent_did}}#keys-1",
                                                                                                                                                               "jws":"{{signature}}"
                                                                                                                                                                 }
                                                                                                                                                                 }

                                                                                                                                                                 Note: sources is optional metadata describing content provenance used to attempt the exercise (e.g., a hash list or source URIs).

                                                                                                                                                                 6.2 Certificate (Aggregated)

                                                                                                                                                                 Human‑readable Markdown referencing all LearningTokens.

                                                                                                                                                                 Contains summary metrics (completion, averages, milestones).

                                                                                                                                                                 Signed by one or more peer validators.


                                                                                                                                                                 Certificate template excerpt (templates/certificate-template.md):

                                                                                                                                                                 # 🎓 SageEduMint – Peer‑Credentialed Certificate

                                                                                                                                                                 **Student:** {{student_name}}  
                                                                                                                                                                 **DID:** {{student_did}}  
                                                                                                                                                                 **Program:** {{program_name}}  
                                                                                                                                                                 **Duration:** {{start_date}} → {{end_date}}

                                                                                                                                                                 ## ✅ Achievements
                                                                                                                                                                 - **Completed Days:** {{days_completed}} / {{total_days}}
                                                                                                                                                                 - **Average Score:** {{average_score}}
                                                                                                                                                                 - **Highlights:**
                                                                                                                                                                 {{#each highlights}}
                                                                                                                                                                 - {{this}}
                                                                                                                                                                 {{/each}}

                                                                                                                                                                 ## 🔗 Verifiable Proofs
                                                                                                                                                                 {{#each tokens}}
                                                                                                                                                                 - Day {{day}} — [Token {{id}}]({{token_uri}}) — Score: {{score}}
                                                                                                                                                                 {{/each}}

                                                                                                                                                                 ## 🪪 Certificate Signature
                                                                                                                                                                 - **Issuer DID:** {{issuer_did}}  
                                                                                                                                                                 - **Signature:** {{certificate_signature}}


                                                                                                                                                                 ---

                                                                                                                                                                 7. Prompts (Key Files)

                                                                                                                                                                 7.1 prompts/daily-content.md (Excerpt)

                                                                                                                                                                 You are a **Content Distiller**. Given syllabus objectives for Day {{day}}, curate and synthesize a concise learning packet.

                                                                                                                                                                 **Constraints**
                                                                                                                                                                 - Prefer open educational resources. Include 5–10 references with inline citations and a reference list.
                                                                                                                                                                 - Summaries must be original; include key definitions, examples, and a 15‑minute quick‑start.
                                                                                                                                                                 - Add a short glossary and 3 checkpoint questions (no answers here).

                                                                                                                                                                 **Output Sections**
                                                                                                                                                                 1. Overview (≤150 words)
                                                                                                                                                                 2. Core Concepts
                                                                                                                                                                 3. Worked Example(s)
                                                                                                                                                                 4. Reference List (links + titles)

                                                                                                                                                                 7.2 prompts/exercise.md (Excerpt)

                                                                                                                                                                 You are an **Adaptive Exercise Generator**. Use the learner’s profile and last 3 evaluations to set difficulty.

                                                                                                                                                                 **Signals**
                                                                                                                                                                 - Mean score last 3 days: {{rolling_score}}
                                                                                                                                                                 - Common error tags: {{error_tags}}

                                                                                                                                                                 **Produce**
                                                                                                                                                                 - 4 tasks: [diagnostic], [core], [extension], [reflection]
                                                                                                                                                                 - Tag each with objective IDs and estimated time.
                                                                                                                                                                 - Provide optional hints gated behind collapsible markers.

                                                                                                                                                                 7.3 prompts/evaluation.md (Excerpt)

                                                                                                                                                                 Evaluate submission against rubric: correctness, clarity, reasoning, and sourcing. Return a JSON block with per‑criterion scores and a 1‑paragraph summary, then a Markdown feedback section for the learner.


                                                                                                                                                                 ---

                                                                                                                                                                 8. CLI Interfaces (Sketch)

                                                                                                                                                                 # Register identity
                                                                                                                                                                 python cli/register.py --name <student>

                                                                                                                                                                 # Create syllabus
                                                                                                                                                                 python cli/generate_syllabus.py --student <student>

                                                                                                                                                                 # Content + exercises for a given day
                                                                                                                                                                 python cli/generate_content.py --student <student> --day 5
                                                                                                                                                                 python cli/generate_exercises.py --student <student> --day 5

                                                                                                                                                                 # Evaluate + issue token
                                                                                                                                                                 python cli/evaluate.py --student <student> --day 5

                                                                                                                                                                 # Adapt syllabus
                                                                                                                                                                 python cli/adapt_syllabus.py --student <student>

                                                                                                                                                                 # Issue certificate
                                                                                                                                                                 python cli/issue_certificate.py --student <student>


                                                                                                                                                                 ---

                                                                                                                                                                 9. Security, Privacy, and Provenance

                                                                                                                                                                 Local‑First Keys: Private keys remain on device; tokens/certificates signed locally.

                                                                                                                                                                 Scraper Policy: Domain allow‑lists, robots.txt respect, and rate limits; artifacts cached locally with integrity hashes.

                                                                                                                                                                 Provenance: Content packets include source lists and optional hash commitments in tokens.

                                                                                                                                                                 Reproducibility: Deterministic prompts + pinned model/version tracked in outputs.

                                                                                                                                                                 Safety & Licensing: Filters to avoid paywalled content and to flag licensing constraints for learners.



                                                                                                                                                                 ---

                                                                                                                                                                 10. Governance & Incentives

                                                                                                                                                                 Peer Validation: Any peer/instructor can issue LearningTokens; validator DIDs are public.

                                                                                                                                                                 Reputation (Future): Validator reputation from cross‑checks and dispute resolution.

                                                                                                                                                                 DAO University (Future): Govern prompt libraries, scraper policies, and credential standards.



                                                                                                                                                                 ---

                                                                                                                                                                 11. Roadmap

                                                                                                                                                                 Phase 1 (MVP)

                                                                                                                                                                 Repo scaffold; CLI for syllabus → content → exercises → evaluation → tokens → certificate.


                                                                                                                                                                 Phase 2

                                                                                                                                                                 PWA dashboard; richer RAG; offline caches + sync; validator reputation metrics.


                                                                                                                                                                 Phase 3

                                                                                                                                                                 DAO governance; decentralized storage (IPFS/Hypercore); torrent‑based model/data distribution; multi‑agent orchestration.



                                                                                                                                                                 ---

                                                                                                                                                                 12. Conclusion

                                                                                                                                                                 SageEduMint University reimagines education as a student‑led, verifiable, decentralized ecosystem. By adding a content generation pipeline and adaptive exercise generator, the learning loop becomes fully autonomous, auditable, and portable — without sacrificing local control or privacy. The outcome is a future‑proof, remixable university that anyone can run, verify, and extend.



                                                                                                                                                                 I’ve added a revised white paper that includes the full content generation pipeline (local agents + web scrapers) and adaptive exercise generation in the end-to-end workflow. If you want, I can also spin up starter code for generate_content.py and generate_exercises.py to match the spec.


                                                                                                                                                                 