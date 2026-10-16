# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.ccpm/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); and rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it).
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

I am the new PM for **Rook Dispatch** (started 24 Aug 2026; I replaced Priya Raghunathan). My director is Helen Achebe. Sources: `rook-wiki` (search_wiki searches titles only; read_page shows comments), `rook-database` (not yet read), and `00-rook/company/notes/handoff-from-priya.docx`. Facts below are as of the wiki's last update, not today (13 Oct 2026).

### Company and products
Rook Industries (founded 2014, HQ Site Aleph, offices in Berlin, Singapore, Cornwall) sells coordination and provisioning software to masked, independent responders and their handlers. Subscription, priced per active responder. Wiki says monthly release train; actual 4.x releases came every 8-10 weeks. Point releases are 4.x.
- **Rook Dispatch** (mine): who gets pinged about a callout, and whether they take it. Users: handlers (console, desktop) and responders (phone).
- **Rook Supply**: gear requisitions, maintenance, failure reports. Users: handlers and quartermasters. Not mine, but it reads Dispatch's Responder Availability Record.
- **Hard rule:** responder cover identities are never stored; no mapping to legal identity exists. Read Security Policy 4.1 before touching responder records. Never try to work out who anyone is.

### People (comments on the wiki use roles, not names)
| Role in comments | Person | Notes |
|---|---|---|
| Engineering manager | Marcus Oyelaran, Site Aleph | Runs Dispatch engineering. Asked the unanswered 4.2 question below. |
| Staff engineer | Wen Li, Berlin | Built the ping-ranking logic. Away 14-24 Aug. Best source on how ranking works. |
| Support lead | Nadia Hoffmann, Berlin | Owns the tickets; has the ticket breakdown ready. |
| Product designer | Sofia Marino, Site Aleph | Owns console and phone app. Ran the September interviews. |
| Data analyst | Ravi Menon, Singapore | Owns the weekly acceptance reports. Priya's handoff says to ask Marcus for numbers; Ravi is the real source. |
| Director of Product | Helen Achebe, Site Aleph | Owns roadmap and commitments. |
Handlers seen in the wiki: Aunt Dot (for Vesper), Mr. Ambrose (Captain Vantage), Halloran (Sgt. Bulwark; also the Supply quartermaster voice), Kip (Meteor Mite, The Gale).

### Vocabulary
- **Callout**: request for a responder at an incident. **Ping**: one callout offered to one responder's phone.
- **Taken / turned down / missed**: yes / no / no answer before the ping wait ends. Turned down and missed are recorded separately, but both move the ping on.
- **Ping wait**: how long a ping stays live. Same for everyone, set per release.
- **Acceptance rate**: share of pings taken. Dispatch's headline metric. **Time-to-accept**: median seconds from sent to taken.
- **Coverage gap**: no available responder had the needed capability tags. Counted apart from low acceptance.
- **Routing priority**: the score that ranks responders. Inputs: proximity (travel-time estimate), availability, capability match, recent acceptance history. Turning down or missing a ping lowers the recent-acceptance part, which lowers later ranking.
- **Capability tags**: flight, structural-entry, hazmat-tolerant, cold-weather, aquatic, crowd-management, de-escalation.
- **Mutual aid**: responders covering for each other across the city. Not supported; Q4 exploration.
- Supply: requisition, field failure report, service interval.

### Where things stand
**Releases:** 4.0 (7 Apr): new nav, profile redesign, routing override audit log. 4.1 (16 Jun): travel-time proximity, bulk callout, push reliability. 4.2 (12 Aug): proximity weighted up vs. acceptance history, ping wait cut 90s to 60s, persistent console filters, three fixes.

**The 4.2 problem (my first job):**
- Since 4.2, callout tickets run about 3x normal (Nadia, 18 Aug). Split roughly two thirds "phone never goes off", one third "gone before I could answer" (26 Aug). Still elevated, flat, as of the last comment (28 Aug).
- The one-third theme is explained by the 60s ping wait. The two-thirds theme has no explanation yet.
- Priya's read: seasonal August dip, will recover in September; do not revert 4.2. The data does not support it (see Findings below). Two changes shipped together, so cause is not isolated.
- Open question from Marcus (14 Aug), never answered: does the ranking change apply to responders who have been turning pings down, or only to others? The config does not distinguish. Wen Li said they would look; no answer is recorded.
- My hypothesis, unverified: a shorter wait turns slow answers into "missed", which lowers recent acceptance, which lowers ranking, so pings stop reaching those responders. Test with Ravi's weekly data, split by before and after 12 Aug and by responder history.
- Marcus asked to regroup on 4.2 once I had a week to look. Nadia will bring the ticket breakdown.

**Roadmap (Q3, last reviewed 30 Jun, all owned by Priya, now stale):**
- 4.2 "Change to who gets pinged": shipped. 4.2 "Ping timeout tuning": a 90s to 60s cut shipped; unclear if "tuning" meant more.
- 4.2 "Availability Confidence": still marked Committed, but not in the 4.2 release notes. Probably squeezed out. Priya said to ask Helen which Q3 items are still commitments. That conversation has not happened.
- 4.3 "Requisition approval chains": Supply item, Committed. Brief bundles a second approval step over $2,000 with at least four other features; scope is undecided.
- Q4, Exploring: Handler phone app (brief 8 Sep; Sofia, unstaffed, scope: notifications, status view, one-tap contact) and Shared cover between responders. Mutual aid is also on the Q4 list.

**Stale briefs:** Routing Override Audit Log (5 Mar) and Bulk Callout (12 Jan, "next quarter") both read as unbuilt but shipped in 4.0 and 4.1. Treat brief status as unreliable; check Releases.

**Gaps I own:** no written description of how ping ranking works (Priya asked me to write one; start with Wen Li). Console filter-persistence tickets are noise; do not spend the first month on them. Bulk callout has no time-to-full-coverage baseline.

### How to help me
Check claims against the wiki and data before repeating them; Priya's handoff is opinion, not fact. Say what is verified and what is a guess. Use role names the way the wiki does, but give me the person's name.

### Findings from the first session (data and code checked)
- Data (`pings`, 29 Jun to 6 Sep): acceptance sat at 75-78% for six weeks, fell to 54% the week 4.2 shipped, and was 72.7% by 31 Aug. Almost all of the drop is missed pings (2-6 a week before, 38 that week). The abrupt start contradicts Priya's "seasonal" view; there is no prior-year data to test "every year".
- Vesper, The Undertow, Meteor Mite and Farlight now get about a third or less of their old ping volume and miss about 59% of what they get (2.9% before). The other 12 responders went from 2% to 14% missed, which fits the 60s wait. Ping position within a callout does not show a ranking slide, because a low rank means rarely pinged, not pinged later.
- Code (`00-rook/code/dispatch-routing`): 4.2 also cut recent-acceptance weight from 0.40 to 0.25 (release notes only mention proximity). A missed ping is scored as a decline (-0.12 vs +0.08 credit), with no decay (2019 TODO). This supports the feedback-loop hypothesis but nobody has tested it.
- Still open: Marcus's 14 Aug question for Wen Li; `support_tickets` not queried (the 3x claim is unverified); Availability Confidence is undefined anywhere; possible Supply knock-on through the availability record; Halloran's interview (approvals too slow) contradicts the requisition brief (add approvals). Releases came Apr, Jun, Aug, not monthly.
- Opinion, not fact: Priya's handoff, Marcus's 26 Aug "August dip", the interview anecdotes (four handlers, one leading question), and Helen's `05-super-speed/director-request.txt`, which assumes the cause is found and the fix is small.

- Module 2 session, tickets (`support_tickets`, 147, 29 Jun to 7 Sep, no category field, many templated): 45 callout tickets after 12 Aug and none before, so the "3x" is wrong (all tickets are about 4.3x). The 2/3 and 1/3 split holds: 30 "phone quiet" (first on 17 Aug, five days after 4.2) and 15 "gone before I could answer" (from 12 Aug). Quiet tickets: Farlight 10, The Undertow 10, Halfmoon 5, Corporal Ashgrove 5. Vesper, Meteor Mite, The Gale and Sgt. Bulwark have none; Aunt Dot, Kip and Halloran filed none. Mr. Ambrose's only ticket (#3043) is his coat-and-one-boot incident, on 12 Aug, not "two weeks" before 2 Sep.
- Pings: the four lost volume over about a week (misses spiked 13-14 Aug first); their turned-down share stayed about 21%. Before 4.2 each area's own responder was pinged on every callout there; after, Vesper, The Undertow and Farlight on about 19-28%, everyone else still on all. Eastgate is shared: Meteor Mite 62 to 9, The Gale 76 to 54. `pings` has no response time and `responders` has only a usual area, so time-to-accept and travel time cannot be tested here.
- Replay of `history.py` on the real outcomes: the four were not low before 4.2 (0.68 or higher), then fall to about 0 by early Sep while the other 12 stay near 0.9. Break-even is 60% taken (0.2 x rate - 0.12); the other 12 took about 60% the first week. Rank gap is up to 0.22 points (about 17 minutes of proximity), but whether neighbors are close enough to be reordered is untested.
- Interviews vs tickets: both agree on gone-before-answer, filter resets, Supply delays and the legend; tickets understate the worst-hit responders. Interviews alone would call quiet "normal"; tickets alone would blame accounts. Phone-alert complaints appear only in interviews.
- Current hypothesis: after 4.2 cut the ping wait from 90s to 60s, responders who answer slowly got missed pings scored as declines with no decay, dropping them below the 60% break-even so they stopped being offered callouts in their own areas. Unverified: that the four were slow before 4.2, and neighbor proximity.
- Next: ask Wen Li or Marcus for per-ping response times and travel-time estimates, plus Marcus's 14 Aug question; ask Sofia to interview Farlight's and The Undertow's handlers (Linda Pruitt, Desmond Okafor); ask Nadia whether Dot, Kip and Halloran report problems another way.

- Module 3 session (pings, 29 Jun to 6 Sep): split at 12 Aug, the 10 Aug week was 72.5% taken before and 46.8% after, and misses start 12 Aug at 09:04. The next ping goes out about 92s after a miss before 4.2 and about 62s after, so the wait cut shows in the data. The four missed 17 of 31 (55%) on 12-16 Aug while their volume was still normal, so slow answers (60 to 90s) most likely started it. Inferred, not measured.
- Replay of `history.py` per responder: all four reach score 0 (The Undertow 27 Aug, Farlight 28 Aug, Vesper 1 Sep, Meteor Mite 5 Sep). The other 12 dip in week one, then recover. From 0 it takes 11 straight takes at 0 to 1 pings a week. A miss counted as nothing would have left the four at 0.76 to 0.84. Restoring the 0.40 weight would make it worse.
- Controls and noise: Meteor Mite and The Gale share handler Kip and area Eastgate, and only Meteor Mite was hit. Before 4.2 the four looked like everyone (75.5% taken vs 77.0%). Halfmoon's and Corporal Ashgrove's "quiet" tickets come from fewer callouts in their areas; both are still pinged on every own-area callout. Pruitt's and Okafor's tickets are templated (identical quote on 1 Sep), so weight `pings` over their wording.
- Tickets ran 5 to 8 a week before and 20, 27, 32, 29 after, while taken % recovered to 73%, so acceptance rate hides the problem. Handlers' "N days quiet" counts match `pings`. Priya's handoff: the seasonal claim is refuted. It is silent on the quiet theme, miss scoring and the weight cut, and "mobile is stable" is untested against the double-notification tickets.
- Hypothesis and fix order: stop scoring a miss as a decline, add decay or a floor (the 2019 TODO), reset the four once, then decide the wait (90s is simplest). Unknown: whether the 60s wait buys anything (no time-to-accept baseline) and travel times. Charts are in `pings-weekly-chart.html` and `replay-scores-chart.html`.
- No area map exists: areas are names only and `travel_time_minutes` is a stub. Neighbors seen in `pings` (Uptown: Cindermark, Sgt. Bulwark, Sgt. Falkirk) took the four's callouts after 4.2 but took none before, which is unexplained. Working rule set today: work alone from wiki, database and code, and never suggest contacting anyone; this replaces the "Next" line above.

- Module 4 session (code read of `00-rook/code/dispatch-routing`, plus `pings`): the flow is `offer.dispatch` -> `routing.rank_for_callout` -> one ping at a time, 60s each, and the first yes ends the search, so a low rank means rarely asked (the README's "nothing decides whether somebody gets asked" misleads). Only two lines change a score: `record_declined` (-0.12, called for turned down AND missed, `offer.py:30`) and `record_accepted` (+0.08). Nothing fades, resets, overrides or credits being asked; the 2019 TODO is the only note. Scores sit in an in-memory dict, so whether a restart or release resets them is unknown.
- Only three values changed in 4.2 (marked "was ... until 4.2" in `config.py`): wait 90s to 60s, proximity weight 0.45 to 0.60, recent-acceptance weight 0.40 to 0.25. The release notes list only proximity; the changelog lists all three. The weight cut softens the damage (a score of 0 vs 0.9 is cancelled by 17 minutes of travel, was 36), so do not restore 0.40. Any other 4.2 logic change could not be checked: no change history in the folder.
- Marcus's 14 Aug question: the weights apply to everyone the same (`score()` never looks at declines). The harm came from misses scored as declines, not from declining; the four's turned-down share stayed near 21%. He asked the day after their misses peaked (my guess, not recorded).
- `pings` by responder, before vs after 12 Aug: the four missed 2.9% then 58.9%; the other 12 went 2.1% to 13.9%. 10-20 Aug: the four missed 17 of 31 on 12-16 Aug while still getting about 6 pings a day, then fell to about 2 a day from 17 Aug. They still took 2 pings on 13 and 14 Aug, which favors slow over dead phone but cannot prove it.
- Replays (real outcomes from 12 Aug, all scores reset to 0.5): a reset alone fails, the four hit 0 again within 6 to 13 pings (14 to 23 Aug), sooner than without a reset, and the other 12 recover. With a miss counted as nothing the four stay at 0.38 to 0.54 and never hit 0, about 8 minutes of travel behind their neighbors; the cost is that a truly absent responder would never lose rank. Not yet run: a half-size miss penalty (0.06), and a fade.
- Still open: restart behavior and where scores are stored; per-ping response times; neighbor travel times; whether any 4.2 logic changed beyond the three values. In this course folder the rule is to work alone and never suggest contacting anyone; I broke it earlier in this session, so check suggestions against it.

- Module 5 session (`05-super-speed/brief.md`, `prototype.html`): the brief ships in two phases, and Helen has not confirmed it. Phase 1 now: wait back to 90s, `SCORE_FLOOR` 0.0 to 0.3, one-time reset of the four to 0.50. Phase 2 next release: first 2 misses in a row free, then -0.06 (needs a miss counter), plus answer-time logging, an optional fade for low scores, and a pass-penalty review.
- Replays apply changed rules to the real `pings` outcomes, so they are lower bounds, and real scores were never observed. A flat 0.06 miss penalty fails. A 0.3 floor alone holds the four at 0.30 on 60s outcomes, but on pre-4.2 90s outcomes they end at 0.84 to 1.00, so the 90s wait does the work. Misses cost about 2.6 seconds of waiting per callout at 90s and about 15 at 60s.
- Scores probably survived the 4.2 release (the four's volume held until 16 Aug), so the reset is a real step. A score of 0 is not a lockout (about 17 minutes of travel; the 0.3 floor is about 11), but a returning hero earns points back slowly. A pass (-0.12) costs more than the first 2 misses (0), with no evidence anyone games that.
- The prototype has 4 pages (4.2 today, after the fix, try it, over time) and no narration inside the product frames. The fast-forward day, answer times and callout lengths are made up. Page 4 projects all 16 responders from real pre-4.2 mixes; the 20% to 100% volume line is my assumption. The student checks it in their own browser (my browser pane can't open it).
- Still open: per-ping response times, whether pushes reach quiet phones, neighbor travel times, the fade design, and whether code outside the folder changed in 4.2.

- Module 6 session (skill and review): `review-checklist` is saved at `.claude/skills/review-checklist/SKILL.md`. It runs six checks (owner, success measure, scope match, problem before fix, sourced claims, definition of ready), gives pass, fail or unclear for each, and ends with a fix list and one next action. It reviews only and never edits the brief. A saved task `monday-review-checklist` runs it on `05-super-speed/brief.md` every Monday at about 9:06 AM; the task file sits outside this folder by my approved exception, and it only runs while the app is open.
- The 4.2 brief scored 2 of 6, then 3 of 6 after I added a definition of ready, a 4-week measurement window (my proposal, Helen to confirm) and "a replay shows" wording for the score of 0. Still open: phase 2 in the status line says only the repeat-miss rule but the body lists four items; no engineering owner; unsourced figures (7 to 2 pings a day, ticket counts, travel minutes).
- Another student's brief (kwarrenIDX) scored 2 of 6. Its "2% to 18% of pings on release day" is wrong: 12 Aug was 28.0% missed, and 18.0% is the average after 4.2. Its 49 to 3 pings a week for the four and Vesper's 70 taken, 5 taken, 9 of 17 missed match `pings`; its "70 of 72" and "5 of 38" area counts cannot be checked, because `pings` has no area.
- Weekly pings for the four were 49, 48, 48, 50, 50, 49 until 9 Aug, then 43, 16, 6, 3 in the weeks of 10, 17, 24 and 31 Aug: the drop took three weeks to bottom out. Misses on 11 to 14 Aug: 5%, 28%, 48%, 27%.
- Working tip: the web fetch tool only returns a summary of a page, so for a brief on GitHub use `curl` on the raw.githubusercontent.com address to get the full text.
