# Rook Industries: course working file

## Session scope, Product School lab

This directory is coursework for Product School's "Codex for PMs"
certification. Everything in it is a fictional teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.codex-for-pms/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it); and, only if the student once said no to trusting this folder, change that one trust setting for this folder in Codex's settings file, keeping a copy of the file first.
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

### Rook and the products

Rook Industries is a fictional coordination and provisioning software company serving independently operating masked responders and the handlers and quartermasters who support them. Rook does not employ responders. Its two products are:

- **Rook Dispatch** coordinates incidents, responder availability, routing and callout offers. Handlers work in the web console; responders use the phone app.
- **Rook Supply** manages equipment requisitions, approvals, fulfillment and maintenance. Handlers and quartermasters use it.

Products ship monthly on a release train; point releases use 4.x numbering. Dispatch writes the **Responder Availability Record**; Supply reads it to schedule maintenance during lower callout periods. Changes to this record can affect Supply even when Supply itself does not change.

### Dispatch: flow, measures and current state

An incident enters the handler console or an intake system. Dispatch first finds available responders who match the incident's required capabilities. If none match, that is a **coverage gap**. Otherwise it ranks the eligible responders, and the handler may override the suggested routing when needed. Dispatch pings the highest-ranked responder and moves down the order if the ping is turned down or missed. A taken ping assigns the incident and marks the responder engaged. If all eligible responders are tried and nobody takes the callout, it is **unfilled**, which is distinct from a coverage gap. Routing configuration ships with releases; handlers cannot adjust it at runtime.

The routing override audit log shipped in 4.0. It records who overrode the recommendation, when, which responder they selected instead, and an optional reason. Check that the log is being used and provides enough context to investigate routing complaints.

The headline measure is **callout acceptance rate**: the share of pings taken rather than turned down or missed. Also track **time-to-accept** (median seconds from ping to take) and **coverage gaps** (incidents with no available responder matching required capabilities). A coverage gap means no eligible person was available; it is different from low acceptance.

The August 12, 2026 release 4.2 increased proximity’s weight relative to recent acceptance history and cut the universal ping wait from 90 to 60 seconds. Since then, fewer pings have been taken and handler complaints have risen. August is seasonally soft, and the timeout changed in the same release, so the impact of the routing change is not isolated. Priya’s handover read was that seasonality may explain much of the decline and advised against reverting the requested routing change without evidence. Check the September trend and separate these factors before drawing conclusions.

The console is stable; the phone app has been stable since 4.1. Routing is the key technical risk and has no adequate written explanation yet. Console filter persistence may create support noise, but Priya considered it cosmetic. The 4.2 roadmap items “Change to who gets pinged,” “Availability Confidence,” and “Ping timeout tuning” were marked committed before shipping; confirm their outcomes. The 4.3 “Requisition approval chains” item is a Supply commitment. “Handler phone app” and “Shared cover between responders” were listed as Q4 exploration. Confirm with Product which items remain Q3 commitments, because that review had not happened during Priya’s handover. Mutual aid is not currently supported and is on the Q4 exploration list.

### People to work with

- **Helen Achebe**, Director of Product: owns roadmap and commitments; clarify which roadmap items remain committed.
- **Marcus Oyelaran**, Engineering Manager, Dispatch: leads Dispatch engineering, gives direct technical feedback, and can usually help with metrics.
- **Wen Li**, Staff Engineer, Dispatch: built the routing decision logic; speak with her to understand responder ranking and document it.
- **Sofia Marino**, Product Designer: owns console and phone app design; ran September interviews.
- **Ravi Menon**, Data Analyst: owns reporting and reports weekly on ping-taking behavior.
- **Nadia Hoffmann**, Support Lead: owns tickets and hears handler pain early; set up a regular check-in.
- **Priya Raghunathan**, former Dispatch PM: left August 21, 2026; her handover is in `00-rook/company/notes/handoff-from-priya.docx`.

### Vocabulary and guardrails

- **Callout**: request to attend an incident. **Ping**: a specific offer of that callout to one responder.
- **Taken**: responder accepts. **Turned down**: responder declines. **Missed**: no answer before the ping wait expires. Both turned-down and missed pings move to the next responder, but remain distinct outcomes.
- **Ping wait**: time before an unanswered ping counts as missed; it is currently the same for everyone and set in a release.
- **Routing priority**: rank for available responders based on travel-time proximity, availability, capability match and recent acceptance history. Declines and missed pings lower the recent-acceptance component.
- **Capability tags** match responder competencies to incident needs. Current examples include flight, structural-entry, hazmat-tolerant, cold-weather, aquatic, crowd-management and de-escalation.
- **Handler** supports one responder or a small group; **quartermaster** manages equipment stock and approvals.

Responder cover identities are never stored in production or mapped to legal identities. Systems hold capability tags, availability windows and callout history. Do not try to infer identities or design features that assume Rook can map them; consult Security Policy 4.1 before changing anything involving responder records.

### First priorities

1. Meet with Helen to confirm roadmap commitments and with Marcus and Wen to understand current routing behavior.
2. Review weekly acceptance, time-to-accept, missed versus declined pings, and coverage gaps with Ravi; compare September with prior Augusts and account for the 60-second timeout change.
3. Establish a regular support check-in with Nadia and use handler feedback to complement the metrics.
4. Write the missing plain-language description of routing, validating it with Wen.

This context comes from the Company wiki pages (About Rook, Rook Dispatch, Rook Supply, Glossary), Team directory, Releases, Q3 roadmap, and Priya’s handover note. Roadmap statuses reflect the last review shown in the wiki, so reconfirm current commitments.

The Rook ping data (June 29–September 6) shows a sharp change around 4.2: the week before release had 134 of 172 pings taken and 2 missed; the week of August 10, which straddles the August 12 release, had 96 of 177 taken and 38 missed. Missed pings remained elevated in the following weeks. This is a signal to investigate, not proof of cause; ranking and ping wait changed together.

Support tickets through September 7 include multiple open reports about offers moving on too soon and responders receiving unusually few offers. These corroborate user concern but do not establish why it is happening. The available release notes still do not identify the three 4.2 defect fixes.
