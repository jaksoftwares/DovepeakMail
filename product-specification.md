DOVEPEAKMAIL
Product Specification, Problem Definition & Technical Architecture
Prepared for Dovepeak Technologies
Version 0.1  |  13 October 2026  |  Status: Initial proposal for technical and commercial validation

Purpose: establish the product scope, operating model, recommended stack, infrastructure requirements, security baseline, commercial hypotheses, and staged implementation plan for a Dovepeak-owned business email hosting service.

 1. Executive Summary
DovepeakMail is proposed as an integrated email platform with two product lines: (1) business email hosting for domain-based mailboxes, and (2) bulk and marketing email campaigns for opted-in customer audiences. Customers should manage domains, users, sending identities, usage, billing and administration from one DovepeakMail account. The platform will share core identity, domain, billing, support and observability services while separating sending streams, reputation controls and policy enforcement. Transactional email and developer sending APIs (OTPs, password resets, receipts, application notifications) are out of scope for DovepeakMail; they are provided by a separate Dovepeak product.
The recommendation is to own the customer-facing platform and automate the email service lifecycle, while using mature mail-server components rather than writing SMTP/IMAP infrastructure from scratch. A VPS can support an internal pilot and a tightly controlled early-access environment, but a single VPS must not be treated as a production-grade, resilient commercial service. Before taking paying customers at scale, Dovepeak must validate outbound deliverability, IP reputation, port 25 availability, abuse handling, backups, recovery, and a second failure domain.
Decision area	Initial recommendation
Product model	Multi-tenant SaaS business email hosting; organisation-based accounts and subscription billing.
Initial customer segment	Kenyan SMEs, agencies, schools, professional firms, nonprofits and growing teams that need domain-based email and straightforward administration.
Infrastructure approach	Linux mail nodes using mature SMTP/IMAP components; separate product control plane from mail transport and storage.
Web application stack	Next.js + TypeScript frontend; NestJS + TypeScript API; PostgreSQL; Redis + BullMQ; S3-compatible object storage for exports and backups.
Email core	Evaluate a maintained mail stack such as Postfix + Dovecot with Rspamd and DKIM signing, or a maintained integrated mail platform. Confirm licensing, operational support and upgrade path before selection.
Launch strategy	Internal Dovepeak deployment → controlled pilot with selected customers → paid launch after deliverability and recovery gates pass.
Commercial principle	Price by mailbox and storage tier, with clear limits, transparent renewals and paid migration/support options.

2. Problem Statement
Businesses need reliable email addresses at their own domains (for example, name@company.co.ke), but setting up and maintaining business email often involves separate providers, confusing DNS changes, manual account creation, limited migration support, unclear storage limits, and fragmented administration. Small organisations may not have a dedicated IT administrator to coordinate domain verification, mailbox setup, access controls, billing, employee offboarding and data migration.
Dovepeak Technologies also wants to reduce dependence on third-party email providers for its own operations and build a service it can commercialise. The challenge is not only to develop an inbox UI; it is to operate a secure, trusted and deliverable email service with strong abuse prevention, customer isolation, reliable backups, support and sustainable unit economics.
2.1 Product opportunity
•	One place to connect domains, verify ownership, configure DNS and manage mailboxes.
•	Self-service organisation administration: users, aliases, groups, roles, policies and offboarding.
•	Migration assistance for customers moving from existing IMAP-based providers and, later, provider-specific APIs.
•	Local-market onboarding and support, including guidance for common DNS providers and Kenyan business customers.
•	A foundation for future Dovepeak services such as domain registration/resale, DNS management, calendars, contacts and team collaboration—kept as separate products or clearly separated services. Transactional email is already served by a separate Dovepeak product.
3. Product Goals and Non-Goals
3.1 Goals
•	Allow a customer to create an organisation, add a domain and prove control of it.
•	Provision mailboxes, aliases, distribution groups and role-based admin access.
•	Send and receive email using webmail and standard clients over secure SMTP submission and IMAP.
•	Guide customers through MX, SPF, DKIM and DMARC configuration with automated checks.
•	Provide storage/quota visibility, account lifecycle controls, audit events and service health visibility.
•	Support mailbox migration with previews, progress, retries, error reporting and final verification.
•	Support plans, subscriptions, invoices/receipts, renewals, suspensions and controlled reactivation.
•	Make backup and restore procedures testable, documented and measurable.
3.2 Non-goals for the first release
•	Building a new SMTP, IMAP or MIME implementation from scratch.
•	Replacing the full Google Workspace or Microsoft 365 productivity suites.
•	Promising enterprise-grade high availability from a single VPS.
•	Launching unrestricted anonymous email sending or using mailbox accounts as a bulk-marketing relay.
•	Building native desktop/mobile clients before a stable webmail and standard-protocol experience exists.
•	Transactional email and developer sending APIs/SMTP relay for application-generated messages; these are provided by a separate Dovepeak product.
•	Treating marketing/bulk email and business mailbox traffic as one undifferentiated sending pool; these are in scope as separate, integrated product modules with distinct controls.
4. Market Reference: How Established Services Organise the Product
The following is a functional benchmark, not a claim that DovepeakMail must reproduce every feature. Current prices and plan entitlements change by region, currency, billing term and promotion; verify them directly before using them in a commercial comparison.
Reference capability	Observed pattern	DovepeakMail implication
Domain onboarding	Add domain → verify ownership → create users/groups → configure MX and mail delivery.	Build a guided setup wizard with DNS instructions, automatic lookup and explicit pass/warn/fail states.
User administration	Create users individually or in bulk; manage aliases, group membership and policies.	Provide mailbox lifecycle actions, CSV import/export, aliases, quotas, disable/reset and offboarding.
Groups	Group addresses distribute mail to selected members and support common organisational addresses.	Support distribution lists first; shared mailbox/delegated access should be a separately specified capability.
Migration	Migration is planned after domains, users and groups exist; IMAP and provider-specific flows may be offered.	Use a migration job model with secure credential handling, incremental sync, retry, mapping and audit logs.
Admin console	Organisation-level control over domains, users, groups, security policies, spam and reports.	Separate customer organisation admin from Dovepeak platform operations and support roles.
Pricing	Per-user tiers with different mailbox storage, attachment limits, collaboration features and support.	Start with simple per-mailbox plans, storage limits, annual/monthly options and clearly stated exclusions.
Security and operations	Business email is more than a mailbox: spam controls, access policies, reports, retention and support matter.	Make abuse prevention, deliverability monitoring, backups, incident response and support part of launch readiness.

Benchmark sources:
•	Zoho Mail domain setup: https://www.zoho.com/mail/help/adminconsole/email-hosting-setup.html
•	Zoho Mail admin console overview: https://www.zoho.com/mail/help/adminconsole/overview.html
•	Zoho Mail user administration: https://www.zoho.com/mail/help/adminconsole/adding-users.html
•	Zoho Mail migration overview: https://www.zoho.com/mail/help/adminconsole/migration.html
•	Zoho Mail IMAP/POP migration: https://www.zoho.com/mail/help/adminconsole/pop-imap-migration.html
•	Zoho Mail pricing comparison: https://www.zoho.com/mail/zohomail-pricing.html
5. Personas and Core User Journeys
Persona	Primary needs	Key journey
Business owner / billing owner	Purchase service, connect domain, control costs, receive invoices.	Register → create organisation → select plan → add domain → pay → monitor subscription.
Organisation administrator	Create users, groups, aliases and security rules.	Verify domain → add mailboxes → configure DNS → invite users → manage changes.
Mailbox user	Read/send email, search, organise messages and use mobile/desktop clients.	Sign in securely → use webmail or IMAP/SMTP client → manage folders and preferences.
Migration lead / IT consultant	Move existing accounts with minimal disruption and clear status.	Create migration → map source/destination → run test → sync → validate → cut over MX.
Dovepeak support operator	Diagnose DNS, delivery, account and billing problems without unrestricted mailbox access.	Review account status/diagnostics → request consent or limited access → act with audit trail.
Dovepeak platform administrator	Manage tenants, plans, mail nodes, abuse cases, billing and platform health.	Monitor queues/health → enforce policies → investigate incidents → suspend or restore service.

5.1 Core onboarding journey (admin-led, comparable to Zoho Mail)
The primary self-service flow is led by one organisation administrator, who sets up the organisation and then brings the team on board. Mailbox users never need to touch DNS, billing or setup.
1.	Sign up: the administrator registers, verifies their own email address, creates the organisation and selects a plan.
2.	Verify domain: the administrator adds the company domain (for example, company.co.ke), publishes the DNS TXT token shown in the portal and the platform confirms ownership.
3.	Create users: the administrator creates mailboxes individually or by CSV import (for example, jane@company.co.ke), or invites members by email; assigns roles, quotas, aliases (info@) and distribution groups (sales@, accounts@).
4.	Activate mail delivery: the setup wizard shows the exact MX, SPF, DKIM and DMARC records, checks them automatically and marks the domain Active once mail routing is correct.
5.	Users sign in: each user accepts their invitation or receives initial credentials, sets a password (and MFA where required), and immediately has a working inbox in webmail and on Outlook, Apple Mail or mobile clients via IMAP/SMTP.
6.	Ongoing administration: the administrator resets passwords, adjusts quotas, adds or removes group members, migrates old mailboxes, and suspends or offboards users who leave, all from the admin console.

6. Functional Requirements
6.1 Identity, organisations and access
•	Customer registration, email verification, sign-in, password reset, session management and MFA for administrators.
•	Organisation/tenant records with an owner, billing contact, timezone, plan, status and verified domains.
•	Role-based access: Owner, Organisation Admin, Billing Admin, Helpdesk/Read-only role, and Mailbox User.
•	Support secure invitations, account recovery, session revocation and administrator audit logs.
•	Support employee offboarding: disable sign-in, revoke sessions, preserve mailbox according to retention policy, transfer/forward mail only with authorised admin action.
6.2 Domains and DNS
•	Add domain and generate a unique TXT verification token; verify through DNS lookup and prevent the same domain from being actively claimed by multiple tenants.
•	Show exact DNS records for ownership verification, MX, SPF, DKIM and DMARC; allow retry and record last-checked timestamps.
•	Check DNS propagation and warn about conflicting MX records, multiple SPF records, missing DKIM, weak DMARC policy and DNS syntax errors.
•	Separate domain ownership verification from mail-routing activation; a verified domain does not automatically mean email is receiving correctly.
•	Maintain a domain status lifecycle: Pending verification, Verified, DNS incomplete, Active, Suspended, Removal pending.
6.3 Mailboxes, aliases and groups
•	Create, edit, suspend, restore and delete mailboxes; assign plan, quota and user identity.
•	Support aliases (for example, info@ mapped to a user) without charging a separate mailbox fee where plan rules allow.
•	Support distribution groups such as sales@, support@ and accounts@ with membership controls and posting permissions.
•	Clearly distinguish an alias, a distribution group and a shared mailbox. Shared mailbox/delegated access requires its own permission model and should not be simulated by sharing passwords.
•	Bulk user creation and updates using validated CSV templates; provide row-level success/error output.
•	Enforce mailbox count, storage, attachment, send-rate and recipient limits per plan and anti-abuse policy.
6.4 Webmail and client access
•	MVP webmail: inbox, sent, drafts, archive, trash, folders/labels, read/unread, search, attachments, compose/reply/forward and contact autocomplete.
•	Support secure IMAP access and authenticated SMTP submission for third-party clients; publish configuration instructions.
•	Responsive interface for desktop and mobile browsers; accessible forms and clear empty/error/loading states.
•	Session security, login alerts and account-level controls. Do not store user passwords in plaintext or expose mail-server credentials in frontend code.
•	Evaluate a maintained webmail client or UI foundation before building every mail interaction from scratch; confirm licence, security maintenance and branding/customisation rights.
6.5 Security, spam and delivery controls
•	TLS for client access and SMTP submission; secure credential storage; MFA for administrators; rate limits and brute-force protection.
•	Inbound spam/virus filtering, reputation checks, outbound rate controls, suspicious-login alerts and abuse reporting.
•	DKIM signing for outbound mail; SPF and DMARC guidance and validation; support TLS reporting and investigate MTA-STS/DANE as maturity increases.
•	Queue visibility, bounce classification, deferred delivery monitoring, blocklist/reputation monitoring and postmaster/abuse workflows.
•	Prevent open relay, compromised-account abuse and unauthorised bulk sending; define acceptable-use policy and response runbooks before public launch.
6.6 Migration and onboarding
•	Initial migration method: IMAP-to-IMAP with encrypted transport, per-mailbox mapping, test connection and progress tracking.
•	Migration job states: Draft, Validating, Queued, Running, Partial, Failed, Completed, Final sync completed.
•	Keep source credentials encrypted only for the shortest required period; restrict access, mask secrets in logs and delete credentials after completion.
•	Support dry-run validation, retries, per-folder/message counts, error exports and final incremental sync before MX cutover.
•	Later integrations: Google Workspace and Microsoft 365 APIs, contacts/calendar migration, PST/mbox/EML import and guided migration assistance.
6.7 Billing and customer support
•	Plans, mailbox entitlements, storage limits, billing period, subscription status, invoices/receipts, renewal dates and payment history.
•	Support manual invoicing/payment reconciliation initially; add a payment gateway after settlement, refund, webhook and reconciliation flows are agreed.
•	Grace period and suspension rules must be explicit. Never silently delete mail immediately when a subscription payment fails.
•	Support ticket or contact form, service status page, documentation, DNS troubleshooting and customer-visible maintenance notices.
7. Administration Surfaces
6.8 Bulk and Marketing Email (Required Product Line)
This capability is in scope for DovepeakMail's all-in-one product vision. It should be implemented as a distinct sending platform connected to the same customer account, domain verification, billing and administration system—not as an unrestricted feature of ordinary user mailboxes.
Marketing/bulk campaigns
•	Manage contact lists, audience segments, tags, custom fields, consent/source records and suppression lists.
•	Create campaigns with a visual or structured editor, reusable templates, preview/test sends, scheduling, timezone-aware delivery and campaign status.
•	Support unsubscribe links, list-unsubscribe headers where applicable, bounce and complaint processing, suppression enforcement and preference management.
•	Track accepted, delivered, deferred, bounced, complained, opened and clicked events where supported; explain that open tracking can be incomplete or privacy-protected by recipient clients.
•	Provide campaign reports, recipient-level event timelines with appropriate access controls, exportable results and campaign audit history.
•	Use queueing, throttling, retries and per-customer limits; support gradual ramp-up and campaign pause/kill controls.
Campaign sending infrastructure and controls
•	Central sending service with durable queues, tenant-level rate limiting, back-pressure, retries, dead-letter handling and idempotent processing.
•	Domain/sender verification and DNS readiness checks for SPF, DKIM and DMARC; support custom return-path/bounce domains where feasible.
•	Event ingestion pipeline that normalises provider/MTA events and deduplicates webhook/event retries.
•	Central suppression service for hard bounces, complaints and unsubscribes; campaigns must not send to suppressed recipients.
•	Customer-level usage metering and billing dimensions for accepted/sent messages, overages, dedicated IPs, storage and optional support.
•	Abuse review, sending reputation monitoring, gradual warm-up guidance, unusual-volume alerts and account suspension workflows.
6.9 Bulk Email Consent and Deliverability Policy
DovepeakMail must distinguish legitimate opt-in marketing from unsolicited bulk sending. Customers must document a lawful basis/consent where required, identify the sender, provide a working unsubscribe mechanism, honour suppression promptly and follow applicable privacy and anti-spam laws. DovepeakMail should prohibit purchased/scraped lists and abusive sending in its acceptable-use policy.
The platform must have independent operational safeguards: sending limits, queue pausing, reputation monitoring, bounce/complaint thresholds, abuse review and separation of campaign traffic from ordinary mailbox traffic. Exact thresholds should be configurable and tuned using pilot data rather than hard-coded without evidence.
Module	MVP scope	Later expansion
Business email hosting	Domains, DNS, mailboxes, aliases, distribution groups, webmail/client access, migration basics.	Shared mailboxes, retention/legal hold options, delegated administration and richer collaboration.
Campaign email	Contact imports, consent/source fields, lists, simple segmentation, templates, test send, scheduling, unsubscribe/suppression, campaign report.	Visual automation journeys, A/B tests, advanced segmentation, landing forms and reusable campaign flows.
Platform operations	Usage limits, tenant quotas, abuse queue, delivery dashboards, billing hooks, service health and audit events.	Automated reputation scoring, more advanced fraud detection, premium SLAs and reseller controls.
Surface	Responsibilities
Customer portal	Signup, organisations, plans, invoices, domains, DNS diagnostics, mailbox inventory, groups, migration jobs, support and usage.
Webmail	Daily mailbox experience; folders, search, compose, attachments, account settings and session/security controls.
Organisation admin console	Users, aliases, groups, policies, quotas, MFA enforcement, offboarding, audit events and domain settings.
Dovepeak operations console	Tenant search, subscription state, mail-node health, queue metrics, abuse reports, support cases, suspensions, incident records and audit review.
Infrastructure/observability	Metrics, logs, alerts, mail queue depth, delivery/bounce rates, storage consumption, backup status, TLS expiry and node capacity.

8. Recommended Technical Architecture
Use a modular control plane that provisions and governs the mail service, with the mail transport/storage tier isolated from the customer web application. The API must not directly expose privileged shell access or accept arbitrary mail-server configuration from users. Provisioning should occur through a narrowly scoped internal service or job worker with validated operations.
Layer	Recommended starting choice	Purpose
Customer UI	Next.js + TypeScript	Marketing pages, onboarding, customer portal and admin interfaces.
API/control plane	NestJS + TypeScript	Tenant, domain, mailbox, plans, billing, audit and provisioning APIs.
Primary database	PostgreSQL	Transactional source of truth for tenants, domains, users, entitlements and jobs.
Queue/cache	Redis + BullMQ	Asynchronous provisioning, DNS checks, migration jobs, notifications and retry workflows.
Mail transfer agent	Postfix	SMTP receiving, outbound queueing, routing and policy enforcement.
Mailbox access/storage	Dovecot plus selected storage layout	IMAP/POP3 access and mailbox delivery/access integration; validate scale and storage design.
Filtering/signing	Rspamd + DKIM signing integration	Spam scoring, rate/policy controls and message authentication.
Webmail	Evaluate Roundcube or another maintained client	Reduce time-to-market; review licence, patch cadence, UX and integration fit.
Object storage	S3-compatible storage for backups/exports; mailbox storage decision separately	Encrypted backups, migration artefacts and export packages.
Edge/security	Reverse proxy, TLS certificates, firewall, fail2ban or equivalent controls	Protect web/API endpoints and reduce unauthorised access.
Observability	Prometheus + Grafana + centralised logs/alerts	Monitor API, queues, mail flow, disk, certificates and backup jobs.
Deployment	Docker Compose for controlled pilot; automated config and backups	Repeatable environments. Do not containerise mail blindly without persistent storage and networking validation.
Sending platform	Dedicated sending API/service + durable queue workers	Orchestrate campaign sending; enforce quotas, retries, idempotency, suppression and event tracking.

Important: Postfix and Dovecot are mature components with documented SMTP and IMAP capabilities, but a production service still requires careful configuration, patching, monitoring and operational expertise. Dovecot Community Edition documentation describes it as designed for a single server; plan for a different architecture or commercial support before assuming multi-node mailbox scale.
Technical references: https://www.postfix.org/documentation.html | https://doc.dovecot.org/main/ | https://doc.dovecot.org/main/core/config/imap.html
8.1 High-level request and mail flow
1.	Customer creates organisation and adds a domain in the DovepeakMail portal.
2.	Control plane issues a DNS verification token; background worker checks authoritative DNS and records the result.
3.	After verification, an organisation admin creates mailboxes/groups. The API validates plan limits and creates an auditable provisioning job.
4.	Provisioning worker updates the mail platform through a restricted integration and confirms the mailbox is ready.
5.	Customer publishes MX/SPF/DKIM/DMARC records; DNS checker verifies the expected state.
6.	Inbound mail arrives at the MX host, passes connection and spam policy checks, then is delivered to the target mailbox.
7.	Outbound mail is submitted through authenticated SMTP, checked against account and tenant sending policies, DKIM-signed, queued and delivered with bounce/defer tracking.
8.	Metrics and logs update operational dashboards; customer-facing delivery diagnostics expose safe, non-sensitive status.
8.2 Logical components
Component	Data / responsibility	Trust boundary
Portal/API	Tenant-facing requests and permissions.	Public internet; strict authentication, validation and rate limits.
Control database	Tenant metadata, entitlements, audit records, mailbox references.	Private network; no direct public access.
Provisioning worker	Applies validated mailbox/domain operations.	Highly privileged; minimal credentials, allowlisted actions and full audit.
Mail edge/MTA	SMTP traffic, queues, reputation and delivery policies.	Internet-facing; hardened, monitored and isolated from portal.
Mailbox service/storage	Mail content, indexes, folders and quotas.	Sensitive data zone; strict tenant/user access and backup controls.
Operations console	Service health and limited support functions.	Restricted staff access, MFA, approvals and audit logging.

9. Data Model: Initial Core Entities
Entity	Key fields / notes
Organisation	id, name, owner_user_id, status, timezone, created_at
User	id, email/login, password_hash or identity-provider reference, MFA state, status
Membership / Role	organisation_id, user_id, role, invitation state
Domain	organisation_id, domain_name, verification_token_hash, verification_state, mail_state
DnsCheck	domain_id, record_type, expected_value, observed_value, status, checked_at
Mailbox	organisation_id, domain_id, local_part, user_id, plan_id, quota_bytes, status
Alias	source address, destination mailbox/group, organisation_id, status
Group	organisation_id, address, posting_policy, membership and moderation rules
Subscription / Plan	plan_id, billing interval, status, renewal date, payment reference
MigrationJob / MigrationAccount	source type, destination mailbox, state, counts, safe error codes, timestamps
AuditEvent	actor, tenant, action, target, timestamp, IP/device metadata, outcome
AbuseCase	report type, affected account/domain, evidence references, status, decisions and timestamps

Email message content should not be stored in the primary business database. Keep mailbox storage in the selected mail-storage layer and store only the identifiers, quota/usage summaries and operational metadata needed by the control plane.
10. Infrastructure and Operational Prerequisites
10.1 Before choosing a VPS or provider
•	Confirm the provider permits email hosting and outbound SMTP on port 25; obtain written confirmation if policy is unclear.
•	Confirm reverse DNS/PTR can be set for the sending IP and that the hostname, PTR and forward DNS can be made consistent.
•	Check whether the IPv4 address is clean and has a credible reputation; avoid shared or recycled IPs with unknown history.
•	Determine IPv6 availability and whether IPv6 outbound mail can be operated with correct PTR, reputation and authentication; disable it for mail if it cannot be configured correctly.
•	Confirm disk IOPS, storage type, backup options, network throughput, bandwidth limits, snapshot consistency and restore support.
•	Use a dedicated hostname such as mail.dovepeakmail.example and separate hostnames for webmail, admin/API and SMTP submission where appropriate.
•	Have a second backup destination outside the VPS and test restoration before accepting customer mail.
10.2 Required DNS and network planning
Record / port	Purpose / policy
MX	Routes inbound email to DovepeakMail's mail host.
A / AAAA	Maps service hostnames; publish only address families that are properly operated.
PTR / reverse DNS	Reverse lookup for outbound mail IP; must align with sending hostname expectations.
SPF (TXT)	Authorises approved sending hosts; publish one coherent SPF record per domain.
DKIM (TXT/CNAME)	Publishes the public key used to verify signed outbound mail.
DMARC (TXT)	Defines alignment policy and reporting; start with monitoring and move to enforcement when legitimate sources are identified.
TLS certificates	Secure webmail, portal, IMAP and SMTP submission; automate renewal and alert on failures.
25 / 465 / 587	SMTP server-to-server delivery and client submission as supported by chosen configuration; document the exact supported modes.
993 / 995	Secure IMAP/POP access if offered; prefer secure client configurations and avoid insecure legacy access.
443	HTTPS for portal, webmail, APIs and status page.

DNS standards and provider-specific record values must be verified during implementation. Do not copy generic sample SPF/DKIM values into production without matching the actual mail hosts and signing configuration.
11. Security, Privacy and Compliance Baseline
•	Tenant isolation: every API query and mail-management operation must be scoped to an organisation and authorised role.
•	Encrypt data in transit; encrypt backups and sensitive stored data at rest where supported. Protect secrets with a secret manager or tightly controlled encrypted configuration.
•	Never log passwords, authentication tokens, full message bodies or migration secrets. Limit diagnostic logging and define retention periods.
•	MFA for all Dovepeak staff and organisation administrators; use least privilege and separate day-to-day accounts from privileged operations accounts.
•	Audit security-sensitive actions: login failures, MFA changes, mailbox creation/deletion, role changes, DNS changes, migration access, suspension and support access.
•	Implement SPF/DKIM/DMARC, spam/virus filtering, outbound quotas, abuse reporting, blocklist monitoring and a documented incident response process.
•	Back up control-plane data, mail data and configuration separately; define recovery point objective (RPO) and recovery time objective (RTO), then run restore drills.
•	Define retention, deletion, export, customer offboarding, legal requests, incident notification and support-access policies before public launch.
•	Obtain Kenyan legal/privacy review, including applicability of the Data Protection Act and ODPC requirements, cross-border processing, contracts, subprocessors and data-subject requests. This document is not legal advice.
•	Publish acceptable-use, privacy, service terms, abuse policy, data processing terms and a clear service-level statement.
12. Deliverability Is a Core Product Requirement
Email deliverability is a launch gate, not a feature to postpone. A technically working SMTP server can still fail commercially if major mailbox providers reject or spam-folder messages. Before production, test inbound and outbound delivery to Gmail, Outlook/Microsoft 365, Yahoo and representative business domains; validate authentication alignment, reverse DNS, TLS, bounce handling, complaint rates, rate limits and IP/domain reputation.
•	Create and monitor postmaster@ and abuse@ operational contacts.
•	Set per-mailbox and per-tenant sending limits; gradually increase legitimate volume instead of allowing sudden bulk sends.
•	Use a clear abuse escalation path with evidence, decision records and an appeal process.
•	Separate marketing/bulk sending from normal mailbox traffic; do not let customer apps use mailbox credentials as a bulk or application email API (application email belongs on Dovepeak's separate transactional product).
•	Maintain mail queue dashboards and alert on unusual queue growth, bounce spikes, deferrals, authentication failures and blocklist listings.
13. Commercial Model and Initial Pricing Hypotheses
The figures below are hypotheses for market testing, not final approved prices. They are deliberately expressed in KES for a Kenya-first launch. Before publication, model VAT/tax, payment fees, domain costs, mail storage, backups, support time, abuse losses, infrastructure redundancy and the cost of acquiring/retaining customers.
Proposed plan	Illustrative price	Possible entitlements
Starter	KES 150–250 / mailbox / month	5–10 GB mailbox, custom domain, webmail, IMAP/SMTP, aliases, standard spam filtering, community/email support.
Business	KES 300–450 / mailbox / month	20–30 GB mailbox, groups, bulk user management, enhanced admin controls, migration tools, priority support.
Business Plus	KES 500–750 / mailbox / month	50 GB mailbox, advanced policies, extended audit/reporting, priority migration assistance and stronger support commitments.
Custom / Enterprise	Quote-based	Custom storage, onboarding/migration project, tailored retention, dedicated support and negotiated service commitments.

Commercial rules to decide: minimum number of mailboxes; monthly versus annual billing; whether aliases and groups are included; storage overage policy; attachment limit; domain registration/resale margin; migration pricing; refund policy; grace period; data export and deletion timeline; support hours; and whether taxes are included in displayed prices.
13.1 Cost model required before setting final prices
•	Fixed monthly infrastructure: mail nodes, control plane, database, backups, monitoring, status page and redundant DNS.
•	Variable costs: mailbox storage, backup retention, outbound bandwidth, payment fees, support time and migration processing.
•	Risk allowance: abuse investigation, compromised accounts, IP reputation recovery, emergency capacity and customer refunds.
•	Target gross margin and support capacity per plan; calculate at 10, 50, 100, 500 and 1,000 mailboxes.
•	Pricing review against current regional Zoho, Google Workspace and Microsoft 365 offers using like-for-like storage, support, billing period and productivity features.
14. Implementation Roadmap
Roadmap note: the first internal proof of concept may run on a VPS, but a commercial all-in-one platform must not depend on a single node for control plane, mailbox data, outbound queues and backups. Separate these failure domains as the service grows, and validate sending throughput, queue recovery and IP reputation before public bulk sending.
Phase	Indicative duration	Deliverables / exit gate
0. Feasibility and decisions	1–2 weeks	Provider/IP/port-25 check; market interviews; licensing review; mail-stack decision; threat model; budget and pricing model. Exit only when delivery feasibility is demonstrated.
1. Internal mail foundation	2–3 weeks	Internal domain, TLS, SMTP/IMAP, SPF/DKIM/DMARC, inbound/outbound tests, backups and restore drill; initial security hardening.
2. Control plane MVP	3–5 weeks	Authentication, organisations, roles, domain verification, DNS diagnostics, mailbox/group provisioning, plans/quotas, audit log and operations console.
3. Webmail and client support	2–4 weeks	Branded webmail integration, responsive UI, SMTP submission/IMAP client setup, mailbox quota and core workflows.
4. Billing and migration	2–4 weeks	Subscription state, invoices/payment workflow, IMAP migration jobs, retries, error exports and cutover checklist.
5. Pilot and reliability	3–6 weeks	Selected pilot customers, delivery matrix, abuse simulations, load tests, restore drills, monitoring, support runbooks and documented incident response.
6. Paid launch	After gates pass	Public plans/terms, support coverage, status page, launch review, capacity limits, operational ownership and rollback plan.
3B. Campaign sending MVP	3–5 weeks (overlapping)	Contact import, templates, usage limits, basic campaigns, unsubscribe/suppression and reporting. Exit only after deliverability and abuse controls pass.

These durations are planning estimates and may overlap where dependencies permit. The schedule must be revised after a technical spike and the team's available capacity are known.
15. MVP Acceptance Criteria
•	A customer can create an organisation and verify control of a domain using a DNS TXT token.
•	The portal correctly detects and explains missing/incorrect MX, SPF, DKIM and DMARC records.
•	An authorised admin can create, suspend and restore a mailbox, alias and distribution group within plan limits.
•	A mailbox can send and receive test messages with supported clients and webmail; authenticated outbound messages pass SPF/DKIM/DMARC alignment tests where configured.
•	Unauthorised cross-tenant access tests fail for all tested organisation, mailbox, group and migration endpoints.
•	Migration jobs provide accurate status, safe error messages, retries and final counts; credentials do not appear in logs.
•	Billing state changes are auditable and do not accidentally trigger immediate irreversible mail deletion.
•	Alerts fire for disk pressure, queue growth, backup failure, certificate expiry, mail service failure and abnormal bounce rates.
•	A clean restore drill meets the agreed RPO/RTO for control-plane data and a documented subset of mail data.
•	Support and abuse runbooks have been exercised, and the launch owner has approved the operational readiness checklist.
•	A simulated bounce spike or abuse event triggers alerting and a documented pause/review workflow.
•	Marketing sending quotas are enforced independently from ordinary mailbox sending.
•	An administrator can sign up, verify a domain, create or invite users, and each user can sign in and send and receive mail from their own inbox without administrator help beyond the invitation.
•	Unsubscribe, complaint and hard-bounce suppression rules prevent future campaign sends to suppressed recipients.
•	An organisation admin can import opted-in contacts, create a campaign, send a test, schedule or pause a campaign, and see campaign-level delivery/bounce results.
16. Key Risks and Mitigations
Risk	Severity	Mitigation
Poor outbound deliverability / IP reputation	Critical	Provider/IP vetting, PTR, SPF/DKIM/DMARC, rate limits, monitoring, staged sending and a pilot delivery gate.
Single VPS failure or disk loss	Critical	Off-host backups, tested restoration, separate backup credentials, capacity alerts and second failure domain before wider launch.
Open relay or compromised mailbox abuse	Critical	Authenticated submission only, outbound quotas, MFA, anomaly detection, abuse response and immediate suspension controls.
Mail data exposure across tenants	Critical	Isolation tests, least privilege, secure storage permissions, restricted support access and audit logging.
Provider blocks SMTP port 25 or restricts mail hosting	High	Verify provider policy and port access before committing; maintain an alternate provider plan.
Scope expands into full collaboration suite	High	Keep calendars, documents, chat and productivity features outside the first release.
Migration failures or data gaps	High	Test migrations, incremental sync, mapping, reconciliation reports, rollback plan and customer sign-off.
Pricing does not cover real operating cost	High	Model storage/support/abuse costs at multiple scales and revise prices before public launch.
Licensing or upstream maintenance concerns	Medium–High	Review licences, update cadence, security advisories and commercial support options for every core component.

17. Preliminary Decisions Required from Dovepeak
•	Confirm whether the first release is business email hosting only or includes domain registration/resale and DNS hosting.
•	Choose the first target segment and initial capacity ceiling (for example, internal use plus a small invited pilot).
•	Approve the technical discovery phase and provider feasibility checks before committing to a mail stack.
•	Choose the first webmail approach after a licence, maintenance and integration review.
•	Agree on customer-facing support hours, backup retention, initial RPO/RTO targets and incident ownership.
•	Approve pricing research and cost modelling before public prices are announced.
•	Define acceptable-use, abuse handling, retention, suspension, export and deletion policies with legal review.
18. Recommended Immediate Work Plan (First 10 Working Days)
Day	Work item	Output
1	Product and market scope workshop	Approved target customer, MVP boundary, customer journey and explicit non-goals.
2	VPS/provider feasibility	Written confirmation on mail hosting, port 25, PTR/rDNS, IP allocation, backup and restore options.
3	Mail-stack evaluation	Decision matrix for integrated mail platform vs Postfix/Dovecot-based stack; licence and upgrade review.
4	Deliverability spike	Working test domain; SPF/DKIM/DMARC; test inbound/outbound to major providers; baseline diagnostics.
5	Threat model and data flow	Trust boundaries, abuse cases, secret handling, tenant isolation and privileged-operation controls.
6	Control-plane schema/API design	Initial ERD, API contracts, roles, domain lifecycle and provisioning job model.
7	Portal wireframes	Domain onboarding, DNS checks, mailbox management, billing and operations-console wireframes.
8	Infrastructure-as-code and CI/CD	Reproducible staging environment, secrets strategy, automated deployment and rollback.
9	Backup and observability setup	Metrics/logs/alerts and a documented, tested backup restoration procedure.
10	Go/no-go review	Updated budget, risks, MVP backlog, milestones and decision to proceed to pilot build.

19. Definition of Done for Initial Preliminary Phase
The preliminary phase is complete when Dovepeak has (1) approved product scope, (2) validated provider and deliverability feasibility, (3) selected a maintainable mail architecture, (4) established a realistic infrastructure and operating budget, (5) documented the threat model and abuse policy, (6) agreed on the first pricing hypotheses, and (7) approved an MVP backlog with acceptance criteria. Do not announce a public launch date before these decisions are made.
20. Reference Links
•	Zoho Mail pricing and editions: https://www.zoho.com/mail/zohomail-pricing.html
•	Zoho domain hosting setup: https://www.zoho.com/mail/help/adminconsole/email-hosting-setup.html
•	Zoho admin console overview: https://www.zoho.com/mail/help/adminconsole/overview.html
•	Zoho adding users: https://www.zoho.com/mail/help/adminconsole/adding-users.html
•	Zoho migration overview: https://www.zoho.com/mail/help/adminconsole/migration.html
•	Zoho IMAP/POP migration: https://www.zoho.com/mail/help/adminconsole/pop-imap-migration.html
•	Postfix documentation: https://www.postfix.org/documentation.html
•	Postfix TLS support: https://www.postfix.org/TLS_README.html
•	Dovecot documentation: https://doc.dovecot.org/main/
•	Dovecot IMAP configuration: https://doc.dovecot.org/main/core/config/imap.html
