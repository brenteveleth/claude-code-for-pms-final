# Brief: fix the 4.2 quiet-responder problem

For: Helen Achebe · From: Dispatch PM · Draft, 16 Oct 2026
Status: proposal. Nothing is built. Engineering has not reviewed it. Helen to confirm: the repeat-miss rule in step 1 and the return to 90s in step 4.

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
2. Optional, later: a fade on old low scores (the 2019 TODO, `history.py:30`). It is not needed to stop the loop. It would let a bad month wear off. A floor of 0.0 already exists (`SCORE_FLOOR`, `config.py:22`). A fade needs a time stamp on each score. An even fade toward 0.5 pulled the other 12's lowest final score from 0.84 to about 0.5, so it should act only on low scores. The rate (I tried 0.03 a day) is a guess, so the brief does not specify one.
3. Reset the four once, only after step 1. A reset alone fails: all four were back at 0 within 6 to 13 pings in the replay. The reset is a real, separate step. Scores probably survived the 4.2 release: if it had wiped them to 0.5, the four would have dropped below 0.3 within 1 to 4 days of 12 Aug, but their volume held at 5 to 9 pings a day until 16 Aug and fell from 17 Aug, which fits scores of 0.84 to 1.00 carrying over. The code holds scores in memory (`history.py:17`), so storage must sit outside this folder. Not proven, because a low score does not drop a responder at once.
4. Restore the 90s ping wait. With the repeat-miss rule, 60s would no longer sink slow responders (the replay used real 60s outcomes), so 90s is not needed to stop the loop. It is still the better default for now: the 60s wait cut no time from callouts in the data. Misses cost about 2.6 seconds of waiting per callout at 90s (25 misses, 879 callouts) and about 15 seconds at 60s (110 misses, 440 callouts). Pings per callout rose from 1.23 to 1.39. The 4.2 weight change also contributes, so the split is not clean. Later: log how long each answer takes, then re-test a shorter wait (60s or 75s) with that data. 75s cannot be predicted from what we have.

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

## Source notes

Verified in code (rechecked 16 Oct against `config.py`, `offer.py`, `history.py`, `routing.py`): wait 60s (was 90), weights 0.60 / 0.25 / 0.15, credit +0.08, penalty -0.12, a miss scored as a decline, no fade, and scores held in memory. The 60% break-even and the 8-minute gap follow from those numbers.
Verified in data: miss rates, volume drop, and the replays (all use real outcomes, so they are a lower bound: with the fix the four get more pings, and how they answer is unknown).
Checked for other 4.2 logic changes: the folder has one commit, so no history. The wiki roadmap rows and the changelog name only the weight change and the wait cut, and the data needs nothing more (neighbors took none of Farlight's callouts before 4.2 only because Farlight took 55 of 59). Code outside this folder could still differ.
Inferred: the four answer slowly.
Opinion, not fact: the handoff doc, "August dip", interview anecdotes.
