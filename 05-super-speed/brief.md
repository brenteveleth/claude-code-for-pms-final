# Brief: fix the 4.2 quiet-responder problem

For: Helen Achebe · From: Dispatch PM · Draft, 16 Oct 2026
Status: proposal. Nothing is built. Engineering has not reviewed it. Helen to confirm: ship in two phases, with phase 1 now (90s wait, one-time reset, 0.3 floor) and phase 2 in the next release (repeat-miss rule).

## The problem in one paragraph

4.2 cut the ping wait from 90 to 60 seconds, so slow answers became "missed". The scoring code counts a miss as a decline (-0.12; a yes gives +0.08) and nothing brings a score back (the 2019 TODO). Low scores mean rarely asked, so a responder cannot earn points back. Four responders (Vesper, The Undertow, Meteor Mite, Farlight) missed 58.9% of pings after 4.2 (2.9% before). A replay of the scoring code on the real outcomes shows all four reached a score of 0 by 5 Sep (real scores were never observed). Their pings fell from about 7 a day (together) to about 2. Priya's "seasonal dip" does not fit: the drop starts on 12 Aug at 09:04.

## 1. Who the fix is for

**Farlight, and Farlight's handler Linda Pruitt.**

- Before 4.2, Farlight was pinged on all 59 callouts in the home area (Uptown) and took 55. After, Farlight was pinged on 6 of 31 (19%) and took 2. Farlight's score reached 0 on 28 Aug.
- Farlight's turned-down share stayed near 21%. What changed was misses, which the score treated as declines.
- Pruitt sees it as "phone quiet" tickets. The wording is templated, so I weight `pings` over the quotes.

Vesper, The Undertow and Meteor Mite follow the same pattern. The other 12 responders also gain from 90s: their misses went from 2.1% to 13.9%, and they dipped in week one, then recovered.

## 2. What changes for them when it ships

**Phase 1, now**
- **Farlight:** the phone goes off again for callouts in Farlight's area. Target: about 1.7 pings a day each for the four, back from about 0.5. Farlight's score restarts at 0.50 and cannot fall below 0.30.
- **Pruitt:** Farlight shows up on own-area callouts again, so "phone quiet" tickets stop. With 90s back, "gone before I could answer" tickets stop for everyone.

**Phase 2, next release**
- A slow responder no longer drops out. The first 2 misses in a row cost nothing, and any real answer resets the count.
- A responder who truly stops answering still slides down, at 0.06 a miss from the third.

## The fix, in two phases

### Phase 1: stop the damage (three small changes)

1. **Restore the 90s wait** (`config.py:9`). This is the load-bearing change. On pre-4.2 outcomes (90s), starting from 0.5, the four never dip below 0.30 and end at 0.84 to 1.00. On the 60s outcomes the same rules leave all four stuck at 0.30. 60s also cut no time in the data: misses cost about 2.6 seconds of waiting per callout at 90s and about 15 at 60s, and pings per callout rose from 1.23 to 1.39 (the weight change also contributes).
2. **Raise the score floor from 0.0 to 0.3** (`SCORE_FLOOR`, `config.py:22`). One number, no new state. It caps the cost of a bad stretch at about 11 minutes of travel, down from about 17. It is a safety net, not a cure: at 60s it only stops the slide.
3. **Reset the four once, to 0.50.** A reset alone fails at 60s (all four were back at 0 within 6 to 13 pings), so it goes with steps 1 and 2. Scores probably survived the 4.2 release (the four's volume held until 16 Aug, which fits scores of 0.84 to 1.00 carrying over, not a wipe to 0.5), and the code keeps them in memory (`history.py:17`), so storage sits outside this folder. Not proven.

### Phase 2: stop it coming back (next release)

1. **Charge only repeat misses.** Today `offer.py:30` scores every answer that is not TAKEN as a decline. Treat MISSED separately: the first 2 misses in a row cost 0, the 3rd and later cost 0.06, and any answer resets the count. This needs a per-responder miss counter. Replay (real outcomes from 12 Aug, scores starting at 0.5): the four stay at 0.32 to 0.54, the other 12 end at 0.84 or higher, and a responder who answers nothing falls from 0.8 to 0.1 in about 14 pings. A flat 0.06 fails (all four reach 0). A flat 0.02 needs no new state: the four stay at 0.22 or higher, and an absent responder takes about 35 pings to go dormant.
2. **Log how long each answer takes.** `pings` has no response time today. Then re-test a shorter wait (60s or 75s). 75s cannot be predicted from what we have.
3. **Optional: fade old low scores** (the 2019 TODO, `history.py:30`). A fade needs a time stamp per score, and should act only on low scores: an even fade toward 0.5 pulled the other 12's lowest final score from 0.84 to about 0.5. No rate is specified (my 0.03 a day was a guess).
4. **Review the pass penalty.** Once misses are free, ignoring a ping can cost less than saying no, and each miss holds the callout for a full wait. Replayed at 0.09 and 0.06: nobody nears 0, but the yes share needed to hold steady drops from 60% to 53% and 43%, which weakens the ranking signal. No evidence heroes do this, so decide with answer-time data.

Considered, not chosen: always asking a home-area hero first. It breaks the loop for good but is a larger routing change.

## How we know it worked

Measured over the 4 weeks after phase 1 ships (window proposed by the PM; Helen to confirm):

- Each of the four is back to about 1.7 pings a day.
- Their miss rate moves toward the others' (about 14% or less).
- Callout tickets fall from 20 to 32 a week back to 5 to 8.
- Acceptance rate is not the test. It recovered to 73% while tickets stayed high.

## Definition of ready

The brief is met when someone other than the author can check each line below.

**Phase 1**
- `OFFER_TIMEOUT_SECONDS` is 90 in `config.py`.
- `SCORE_FLOOR` is 0.3 in `config.py`.
- No other value in `config.py` changed: weights 0.60 / 0.25 / 0.15, credit +0.08, penalty -0.12.
- Right after the reset, `history.recent_acceptance` returns 0.50 for Vesper, The Undertow, Meteor Mite and Farlight.
- After the reset, no responder's score is below 0.30.

**Phase 2**
- A test shows the first 2 misses in a row change a score by 0, and the 3rd and later each cost 0.06.
- A test shows a taken or turned-down answer resets the miss count to 0.
- Every new ping row has an answer time (seconds from sent to answered). Rows for missed pings say so.
- The one-time reset from phase 1 is not run again.

## 3. What the fix does not do

- **It does not put Farlight first.** Farlight still sits about 8 minutes of travel behind neighbors after the reset, so a neighbor may be asked first. That is the 4.2 proximity change working as intended.
- **It does not stop the next slow stretch (until phase 2).** In phase 1 a miss still costs 0.12. The floor caps the damage at 0.30, but a hero who misses a lot will sit there until they answer more.
- **It does not make a silent responder vanish.** Ranking never removes anyone. At the floor a hero costs about 11 minutes of travel and is still asked when no neighbor is close, and each miss costs the callout a full wait.
- **It does not prove Farlight was slow.** There is no response-time data, and the four took only 18% to 29% of pings after 12 Aug while missing 53% to 64%. Outcomes alone cannot tell slow from absent. A push that arrives only some of the time would also cause "phone never goes off". Untested.
- **It does not prove 60s was pointless.** There is no time-to-accept baseline, so 90s is the simplest move, not a proven one.
- **It does not undo other 4.2 changes.** The proximity weight stays (0.45 to 0.60, effect on neighbors untested). The acceptance weight stays at 0.25, because restoring 0.40 would make it worse. 4.2 is not reverted.
- **It does not touch:** console filters, Availability Confidence, mutual aid, or the Supply link through the availability record (a possible knock-on, untested).
- **It does not give a new screen.** Handlers see pings returning, not a new feature.

## Source notes

Verified in code (rechecked 16 Oct against `config.py`, `offer.py`, `history.py`, `routing.py`): wait 60s (was 90), weights 0.60 / 0.25 / 0.15, credit +0.08, penalty -0.12, a miss scored as a decline, no fade, scores held in memory. The 60% break-even and the 8-minute gap follow from those numbers.
Verified in data: miss rates, volume drop, and the replays. The replays use real outcomes, so they are a lower bound: with the fix the four get more pings, and how they answer is unknown.
Other 4.2 logic changes: the folder has one commit, so no history. The wiki roadmap rows and the changelog name only the weight change and the wait cut, and the data needs nothing more. Code outside this folder could still differ.
Inferred: the four answer slowly. Opinion, not fact: the handoff doc, "August dip", interview anecdotes.
