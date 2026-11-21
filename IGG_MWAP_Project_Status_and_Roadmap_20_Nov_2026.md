**IGG Job Monitor**

# **MWAP Project Status** **& Development Roadmap**

A review document for WhatsApp Community administrators

**Prepared: 20 November 2026  \|  Development and validation phase**

# **1\. Executive summary**

IGG Job Monitor (MWAP) is being developed to discover videogame-industry vacancies in India, verify that listings are genuine and active, prevent duplicates, route suitable jobs to the correct WhatsApp Community groups, and support controlled distribution.

The system uses Google Sheets as its central database and a standalone Google Apps Script project as its automation engine. The approval workflow has passed controlled end-to-end tests using temporary records. Discovery and database-writing components have been developed and tested in read-only or controlled ways, but production collection and genuine vacancy insertion have not been activated.

Automated posting to existing WhatsApp Community groups is a desired future capability, not a completed feature. The current investigation is focused on a locally hosted, unofficial WhatsApp Web automation gateway (WAHA) using the existing personal WhatsApp number. This carries account, security and reliability risks.

# **2\. Project objectives and scope**

* Discover vacancies across videogame studios and publishers, external QA/testing providers, art and animation outsourcing, localisation providers, gaming technology vendors and other videogame-industry service providers.  
* Include casino, gambling and real-money gaming employers where the employer, role and India-eligibility checks are met.  
* Filter vacancies by employer eligibility, job discipline, India eligibility and freshness.  
* Verify vacancy availability using vacancy-specific evidence; a successful webpage response alone is insufficient.  
* Detect duplicates and preserve source history and traceability.  
* Route jobs using Job\_Discipline and approved group mappings.  
* Require human approval before a job becomes eligible for distribution.  
* Prepare distribution records and, subject to technical validation, automate posting to the correct existing WhatsApp Community group.

# **3\. Current system architecture**

| Component | Role / current position |
| :---- | :---- |
| Google Sheets | Central database: jobs, sources, companies, categories, group mappings, approval queue, distribution log, configuration and audit/error records. |
| Google Apps Script | Standalone automation engine connected to the spreadsheet by ID; contains discovery, verification, duplicate, database preview/writer and approval workflow code. |
| ASGC job source | Integration developed against the ASGC job-listings endpoint. Source SRC001 remains disabled pending terms/permission review. |
| Approval workflow | Processes initial and revised decisions, updates JOBS\_MASTER, records audit history and requires explicit confirmation for changed decisions. |
| WhatsApp integration | Not connected. WAHA Core is being considered for local testing with the existing personal WhatsApp account. |

# **4\. Functions developed and current readiness**

| Area | Developed capability | Status / qualification |
| :---- | :---- | :---- |
| Spreadsheet foundation | Structured tabs for jobs, sources, employers, categories, group mapping, approvals, distribution, configuration and logs. | Implemented; database structure established. |
| Employer register | Approved employer register and employer-decision process. New promotions should start disabled pending review. | Implemented; project baseline recorded 98 approved unique employers. |
| Job discovery | ASGC API intake, read-only preview, source posting IDs and source history. | Development integration tested; source disabled pending permission/terms review. |
| Eligibility filtering | Employer, role/title and India relevance checks, including explicit exclusions. | Implemented in intake workflow; production collection inactive. |
| Vacancy verification | Dedicated verification for Workday, Greenhouse, SmartRecruiters and EA, plus generic fallback. Active requires positive vacancy-specific evidence. | Platform routing tests 9/9 and real-job tests 6/6 previously passed. Unsupported/inconclusive platforms may remain unverified. |
| Duplicate detection | Employer \+ requisition ID and employer \+ specific application URL checks; title similarity and location compatibility for possible duplicates. | Implemented; in-memory tests 4/4 and database lookup scenarios 8/8 passed. Some source-identity conflict scenarios deferred. |
| Database preview | Prepares 32-column job records and 11-column source records without writing genuine jobs. | Tested; latest recorded preview prepared eight eligible records with structural audits passing. No production insertion. |
| Production writer | Locking, fresh duplicate checks, permanent IDs, source-history linkage and rollback attempt if source writing fails. | Code developed; safety guard blocked commit while emergency pause active. Not production-activated. |
| Approval queue builder | Selects active, pending-review, not-ready jobs; exact Job\_Discipline routing; holds missing/conflicting mappings; avoids duplicate queue entries. | Routing tests 5/5; row builder tests 10/10. Empty-queue execution tested; genuine queue insertion not yet validated. |
| Approval decision rules | PENDING, APPROVED, REJECTED and HOLD; publishing is separate from approval. | Decision tests 7/7 passed. |
| Approval processing | Initial decisions applied automatically; changed decisions require Reprocess\_Confirmed=TRUE; revisions update master and audit log. | Processing tests 11/11 passed. Controlled live test validated initial approval, blocked revision, confirmed revision, audit trail and confirmation reset. |
| WhatsApp posting | Automated delivery to mapped Community groups is the target capability. | Not built. WAHA local feasibility test is the next technical investigation. |

# **5\. Approval workflow: validated behaviour**

The workflow was tested end to end using temporary records, which were subsequently cleaned up.

* An initial APPROVED decision updates JOBS\_MASTER.Review\_Status and records an INITIAL\_DECISION audit entry.  
* An unchanged decision produces no additional action.  
* A changed decision without confirmation is marked REPROCESSING\_REQUIRED; the master record and audit log are not changed.  
* Setting Reprocess\_Confirmed to TRUE permits the revised decision to be applied.  
* A successful revision records Previous\_Decision, New\_Decision and DECISION\_REPROCESSED in APPROVAL\_AUDIT\_LOG, then resets the confirmation flag to FALSE.  
* Approval does not publish or send a WhatsApp message; Publish\_Status remains separate.

The controlled test exposed two property-reference defects in the live processor. Both were corrected, and a subsequent confirmed revision test passed. Temporary test records and their associated audit entries were removed after validation.

# **6\. Safety controls and current operating state**

| Control | Current setting / rule |
| :---- | :---- |
| EMERGENCY\_PAUSE | TRUE — keep unchanged during development. |
| MONITORING\_ENABLED | FALSE — no scheduled monitoring. |
| OPERATING\_MODE | APPROVAL\_REQUIRED. |
| ASGC source SRC001 | Enabled \= FALSE; manual development integration only. |
| Production collection / insertion | Not activated; no genuine vacancies inserted through the production writer. |
| WhatsApp posting | No WhatsApp account connected; no MWAP distribution performed. |
| Human approval | Required before distribution. |
| Traceability | Preserve employer identity, source references and duplicate safeguards. |

The system is not operational or production-ready. Component-level tests and controlled workflows do not constitute a complete live end-to-end job-to-WhatsApp service.

# **7\. Next functions to build**

| Step | Workstream | Planned work / completion criteria |
| :---- | :---- | :---- |
| 10.1 | Local WhatsApp feasibility | Install Docker Desktop and WAHA Core locally; connect existing personal number through linked-device authentication; inspect group list without sending messages. |
| 10.2 | Community group compatibility | Verify whether existing Community subgroups appear and can be addressed. Ordinary group support does not prove Community subgroup support. |
| 10.3 | Controlled message test | After explicit authorisation, send one labelled test message to a designated test group; verify delivery and capture group identifier. |
| 10.4 | Secure Apps Script bridge | Design secure communication from Apps Script to local WAHA. Do not expose the WAHA interface publicly without authentication and access controls. |
| 10.5 | Distribution eligibility engine | Allow only active, verified, approved and successfully processed jobs; require exactly one enabled Job\_Discipline mapping; hold missing/conflicting routes. |
| 10.6 | Message formatter | Prepare consistent messages with title, employer, location, work arrangement, experience, discipline and original application URL. |
| 10.7 | Automated posting and duplicate prevention | Send only after safeguards pass. Record destination, attempt, response/message identifier, timestamp and outcome; prevent repeat sends. |
| 10.8 | Distribution tracking | Complete DISTRIBUTION\_LOG for sent, failed, skipped and held jobs; support controlled retries without duplicate posts. |
| 11 | Monitoring and admin reporting | Show posted jobs, failed sends, pending approvals, routing conflicts and source/verification issues. |
| 12 | Scheduling and end-to-end validation | Only after earlier stages pass: design morning processing, recovery, monitoring and operational handover. Production activation requires separate approval. |

# **8\. WhatsApp automation: approach and risks**

The current candidate is WAHA Core, a self-hosted HTTP API gateway that operates through a linked WhatsApp Web session. The proposed first test is local on the user's Windows computer. Docker Desktop is the recommended installation route. No VPS or paid hosting is needed for the initial local test.

* The existing personal WhatsApp number is intended to be retained; no additional number is planned.  
* WAHA is unofficial and not endorsed by WhatsApp/Meta. Automated access and messaging may violate platform terms and may lead to account restrictions or bans.  
* Self-hosting does not remove account, session, security or protocol-change risks.  
* The linked-device session and API credentials must be protected. Do not expose the service publicly without appropriate authentication and network controls.  
* Local operation depends on the computer, internet connection and session being available at posting time.  
* A VPS may be considered later for continuous availability, only after local compatibility and delivery are validated and hosting costs are approved.

The WAHA test should begin read-only: connect the account and inspect available groups. No test message should be sent to an actual distribution group without explicit approval.

# **9\. Decisions and feedback requested from Community administrators**

* Do administrators support automated posting of approved videogame-industry vacancies into the existing Community groups?  
* Should posts be sent immediately after approval, or should there be an additional administrator release/hold step?  
* Are there group-specific posting rules, quiet hours, daily limits or content restrictions?  
* Should each vacancy be routed to only one group, or can some vacancies be shared across multiple groups?  
* What should happen when a vacancy is updated, reposted, expires or is later found invalid?  
* Who should be authorised to approve jobs, confirm revised decisions, pause posting and review failures?  
* Are administrators comfortable with the risks of an unofficial automation method using a personal WhatsApp account, or should the project stop at manual distribution if those risks are unacceptable?

# **10\. Current project position**

The core database, employer controls, discovery/verification development, duplicate safeguards, preview pipeline, approval queue builder and approval workflow have been substantially developed. The approval workflow is functionally validated. Source activation, production insertion, genuine vacancy queue generation, WhatsApp integration, distribution logging and full production end-to-end testing remain outstanding.

The immediate next milestone is a local WAHA feasibility test using the existing personal WhatsApp number and a designated test group. No hosting purchase, production activation or automated posting should occur until technical testing, administrator feedback and the risk decision are complete.

**Document purpose:** For review and feedback by the remaining WhatsApp Community administrators. This report is not an approval to activate production automation.