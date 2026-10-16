# Brief: fix the 4.2 quiet-responder problem

For: Helen Achebe · From: Dispatch PM · Draft, 16 Oct 2026
Status: proposal. Nothing is built. Engineering has not reviewed it.

## The problem in one paragraph

4.2 cut the ping wait from 90 to 60 seconds. Slow answers became "missed". The scoring code counts a miss as a decline (-0.12; a yes gives +0.08), and nothing ever brings a score back (the 2019 TODO). Low scores mean rarely asked, so a responder cannot earn points back. Four responders (Vesper, The Undertow, Meteor Mite, Farlight) missed 58.9% of pings after 4.2 (2.9% before). All four reached a score of 0 by 5 Sep. Their pings fell from about 7 a day (all four together) to about 2.

Priya's "seasonal dip" does not fit this. The drop starts on 12 Aug at 09:04.

## 1. Who the fix is for

**Farlight, and Farlight's handler Linda Pruitt.**

- Before 4.2, Farlight was pinged on every callout in the home area. After, Farlight got about a third or less of them. Farlight's score reached 0 on 28 Aug. (The four together went from about 7 pings a day to about 2.)
- Farlight did nothing different. The turned-down share stayed near 21%. The score fell because misses were scored as declines.
- Pruitt is the person who sees it. Pruitt files "phone quiet" tickets. (The wording is templated, so I weight `pings` over the quotes.)

Same pattern, same fix: Vesper, The Undertow, Meteor Mite.

Second group: the other 12 responders. Their misses went from 2.1% to 13.9%. They dipped in week one, then recovered. They gain from the 90s wait ("gone before I could answer").

## 2. What changes for them when it ships

For Farlight:
- The phone goes off again for callouts in Farlight's area. Target: about 1.7 pings a day each for the four, back from about 0.5.
- A missed ping stops counting as a decline. Farlight's score stays at 0.38 to 0.54 in my replay and never reaches 0.
- Farlight's scores get a one-time reset, after the scoring fix. Not before.

For Pruitt:
- Farlight shows up on own-area callouts again. "My responder has gone quiet" tickets stop.
- With 90s back, "gone before I could answer" tickets stop for everyone.

For everyone:
- A responder who is slow for a while no longer drops out. With a fade or floor, a responder who truly stops answering still slips down, only slowly.

## The fix, in order

1. Stop scoring a miss as a decline. In `offer.py:30` every answer that is not TAKEN calls `record_declined`, and `offer_to` returns TURNED_DOWN or MISSED. The change is to treat MISSED differently there. (A 0.06 miss penalty is a safer middle. Not yet replayed.)
2. Add a fade, or a higher floor, to scores (the 2019 TODO, `history.py:30`). A floor of 0.0 already exists (`SCORE_FLOOR`, `config.py:22`). It is why the four sit at 0 and cannot go lower. A fade needs a time stamp on each score. Today a score is a bare number. Design not chosen.
3. Reset the four once, only after steps 1 and 2. A reset alone fails: all four were back at 0 within 6 to 13 pings in the replay.
4. Restore the 90s ping wait.

Fast stopgap: step 4 plus the reset. It stops new misses but leaves the loop for the next slow responder.

## How we know it worked

- Each of the four is back to about 1.7 pings a day.
- Their miss rate moves toward the others' (about 14% or less).
- Callout tickets fall from 20 to 32 a week back to 5 to 8.
- Acceptance rate is not the test. It recovered to 73% while tickets stayed high.

## 3. What the fix does not do

- **It does not put Farlight first.** In the replay Farlight still sits about 8 minutes of travel behind neighbors, so a neighbor may be asked first. That is what the 4.2 proximity change meant.
- **It does not prove Farlight was slow.** There is no response-time data. "Slow answers" is inferred from the timing of the misses.
- **It does not explain a dead push.** A push that arrives only some of the time would also cause "phone never goes off". Untested.
- **It does not remove all risk from the miss rule.** If a miss counts as nothing, a responder who has really gone away never loses rank. Step 2 covers this, but only once designed.
- **It does not say what the 60s wait was for.** There is no time-to-accept baseline. Restoring 90s is the simplest move, not a proven one.
- **It does not restore the 0.40 weight on recent acceptance.** That would make it worse. It does not undo the 0.45 to 0.60 proximity change, whose effect on neighbors is untested.
- **It does not revert 4.2.** The proximity change stays.
- **It does not touch:** console filters, Availability Confidence, mutual aid, or the Supply link through the availability record (a possible knock-on, untested).
- **It does not give a new screen.** This is a behavior fix. Handlers see it as pings returning, not as a new feature.

## Open before we build

- Are scores saved, or wiped on restart? In this code they sit in an in-memory dict (`history.py:17`), so a restart sends everyone back to 0.5. If that is how production works, shipping any release would reset everyone and look like a fix for a while, then the loop would start again. The reset step would also happen on its own. The folder has no code that saves scores, but it only shows stubs for the rest of the service.
- Did 4.2 change any logic beyond the three marked values (wait, proximity weight, acceptance weight)? No change history is in the folder.
- Which fade or floor, and what miss penalty?

## Source notes

Verified in code (rechecked 16 Oct against `config.py`, `offer.py`, `history.py`, `routing.py`): wait 60s (was 90), weights 0.60 / 0.25 / 0.15, credit +0.08, penalty -0.12, a miss scored as a decline, no fade, and scores held in memory. The 60% break-even and the 8-minute gap follow from those numbers.
Verified in data: miss rates, volume drop, both replays.
Inferred: the four answer slowly.
Opinion, not fact: the handoff doc, "August dip", interview anecdotes.
