# Brief: fix the 4.2 quiet-responder problem

For: Helen Achebe · From: Dispatch PM · Draft, 16 Oct 2026
Status: proposal. Nothing is built. Engineering has not reviewed it.

## The problem in one paragraph

4.2 cut the ping wait from 90 to 60 seconds. Slow answers became "missed". The scoring code counts a miss as a decline (-0.12; a yes gives +0.08), and nothing ever brings a score back (the 2019 TODO). Low scores mean rarely asked, so a responder cannot earn points back. Four responders (Vesper, The Undertow, Meteor Mite, Farlight) missed 58.9% of pings after 4.2 (2.9% before). All four reached a score of 0 by 5 Sep. Their pings fell from about 7 a day (all four together) to about 2.

Priya's "seasonal dip" does not fit this. The drop starts on 12 Aug at 09:04.

## 1. Who the fix is for

**Farlight, and Farlight's handler Linda Pruitt.**

- Before 4.2, Farlight was pinged on all 59 callouts in the home area (Uptown) and took 55. After, Farlight was pinged on 6 of 31 (19%) and took 2. Farlight's score reached 0 on 28 Aug. (The four together went from about 7 pings a day to about 2.)
- Farlight did nothing different. The turned-down share stayed near 21%. The score fell because misses were scored as declines.
- Pruitt is the person who sees it. Pruitt files "phone quiet" tickets. (The wording is templated, so I weight `pings` over the quotes.)

Same pattern, same fix: Vesper, The Undertow, Meteor Mite.

Second group: the other 12 responders. Their misses went from 2.1% to 13.9%. They dipped in week one, then recovered. They gain from the 90s wait ("gone before I could answer").

## 2. What changes for them when it ships

For Farlight:
- The phone goes off again for callouts in Farlight's area. Target: about 1.7 pings a day each for the four, back from about 0.5.
- One or two missed pings in a row stop costing Farlight anything. Only the third miss in a row and later ones cost points (0.06 each), and any answer resets the count. In my replay Farlight's score stays at 0.36 or higher and never reaches 0.
- Farlight's scores get a one-time reset, after the scoring fix. Not before.

For Pruitt:
- Farlight shows up on own-area callouts again. "My responder has gone quiet" tickets stop.
- With 90s back, "gone before I could answer" tickets stop for everyone.

For everyone:
- A responder who is slow for a while no longer drops out, because slow answers are mixed with real answers and reset the miss count.
- A responder who truly stops answering still goes dormant: about 14 pings in a row with no answer takes a score from 0.8 to 0.1.

## The fix, in order

1. Stop scoring a miss as a decline, and charge only repeat misses. In `offer.py:30` every answer that is not TAKEN calls `record_declined`, and `offer_to` returns TURNED_DOWN or MISSED. The change is to treat MISSED differently there: the first 2 misses in a row cost 0, the 3rd and later cost 0.06, and any answer resets the count. This needs a miss counter per responder, which does not exist today. Replayed 16 Oct on real outcomes from 12 Aug, scores starting at 0.5: the four stay at 0.32 to 0.54 and never reach 0, the other 12 end at 0.84 or higher, and a responder who answers nothing falls from 0.8 to 0.1 in about 14 pings. A flat 0.06 miss penalty fails (all four reach 0). A flat 0.02 is the option with no new state: the four stay at 0.22 or higher, and an absent responder takes about 35 pings to go dormant.
2. Optional, later: a fade on old low scores (the 2019 TODO, `history.py:30`). It is not needed to stop the loop. It would let a bad month wear off. A floor of 0.0 already exists (`SCORE_FLOOR`, `config.py:22`). A fade needs a time stamp on each score. Design open (see below).
3. Reset the four once, only after step 1. A reset alone fails: all four were back at 0 within 6 to 13 pings in the replay.
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
- **It does not tell slow from absent in the data.** After 12 Aug the four took only 18% to 29% of pings and missed 53% to 64%. The repeat-miss rule is built to protect a slow responder, but outcomes alone cannot prove which one Farlight is. The 90s wait is what fixes slowness.
- **It does not make an absent responder vanish.** Ranking never removes anyone. At the 0.25 weight a score of 0 costs only about 17 minutes of travel, so a dormant responder is still asked when no neighbor is close, and each miss costs the callout one full wait.
- **It does not say what the 60s wait was for.** There is no time-to-accept baseline. Restoring 90s is the simplest move, not a proven one.
- **It does not restore the 0.40 weight on recent acceptance.** That would make it worse. It does not undo the 0.45 to 0.60 proximity change, whose effect on neighbors is untested.
- **It does not revert 4.2.** The proximity change stays.
- **It does not touch:** console filters, Availability Confidence, mutual aid, or the Supply link through the availability record (a possible knock-on, untested).
- **It does not give a new screen.** This is a behavior fix. Handlers see it as pings returning, not as a new feature.

## Open before we build

- Are scores saved, or wiped on restart? In this code they sit in an in-memory dict (`history.py:17`), so a restart sends everyone back to 0.5. If that is how production works, shipping any release would reset everyone and look like a fix for a while, then the loop would start again. The reset step would also happen on its own. The folder has no code that saves scores, but it only shows stubs for the rest of the service. Data check, 16 Oct: if the 4.2 release had wiped scores to 0.5, the four would have dropped below 0.3 within 1 to 4 days of 12 Aug. Their volume held at 5 to 9 pings a day until 16 Aug, then fell from 17 Aug. If scores carried over (0.84 to 1.00 on 12 Aug), they cross 0.3 between day 5 and day 11, which fits better. So scores probably survived the release. That points to storage outside this folder, and it means the one-time reset is a real, separate step. Not proven: a low score does not drop a responder at once, because the rank depends on neighbors.
- Did 4.2 change any logic beyond the three marked values (wait, proximity weight, acceptance weight)? Checked 16 Oct: the folder has one commit, so no history. The wiki roadmap rows for 4.2 only say "weight proximity more heavily" and "tune the ping wait". The changelog lists the same two changes. The data needs nothing more: in Uptown, neighbors took none of Farlight's callouts before 4.2 only because Farlight took 55 of 59. After 4.2 Farlight is rarely asked, so neighbors take them. Not provable: code outside this folder could still differ.
- Decided 16 Oct (my recommendation, for Helen to confirm): the first 2 misses in a row cost 0, the 3rd and later cost 0.06, any answer resets the count. Fallback if engineering wants no new state: a flat 0.02 per miss. A flat 0.06 fails in the replay, and a fixed floor is too blunt.
- Not decided: whether to add a fade at all. It is optional, and a second step. An even fade toward 0.5 pulled the other 12's lowest final score from 0.84 to about 0.5, so it should act only on low scores. The rate (I tried 0.03 a day) is a guess.

## Source notes

Verified in code (rechecked 16 Oct against `config.py`, `offer.py`, `history.py`, `routing.py`): wait 60s (was 90), weights 0.60 / 0.25 / 0.15, credit +0.08, penalty -0.12, a miss scored as a decline, no fade, and scores held in memory. The 60% break-even and the 8-minute gap follow from those numbers.
Verified in data: miss rates, volume drop, and the replays (all use real outcomes, so they are a lower bound: with the fix the four get more pings, and how they answer is unknown).
Inferred: the four answer slowly.
Opinion, not fact: the handoff doc, "August dip", interview anecdotes.
