# 04 · X-Ray Vision — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. Last session the numbers showed you what happened: a
ping used to wait ninety seconds and now it waits sixty, people
missed pings they used to catch, and missing one counts the same as
turning one down — so four responders stopped hearing from us
altogether.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

check the weight cut on line 14

### 2.

now, complete this sentence:

"Based on what I found, the reason some responders are getting no pings at all is ___, because ___."

### 3.

Are there any engineering comments in the codebase worth evaluating further

### 4.

what code changes were introduced in 4.2 that may be contributing towards the issues we've been seeing with more misses and less pings being sent

### 5.

Who is our engineering manager?

### 6.

Query pings for misses by responder before and after 12 Aug

### 7.

alongside the code, does this back up what we've found so far?

### 8.

Query the four's misses day by day for 10 to 20 Aug

### 9.

Answer Marcus' question in one sentence

### 10.

BTW why might Marcus be asking his question?

### 11.

what takes points off somebody, then find everything that puts them back

### 12.

Does the service restart on release, and would that reset scores?

### 13.

Does anything in the code reset a score?

### 14.

Even if we were to reset all the score, starting all responders fresh, what do you think would happen? Would you expect the same trends, or something different?

### 15.

Replay a reset on 12 Aug

### 16.

Now, with what we know, complete this sentence:

According to my findings in the code, for someone who's gone quiet, they would need to ___.

### 17.

run the same replay with a miss counted as nothing instead of a decline. That tests whether the scoring rule alone is enough, and takes about five minutes.

### 18.

If I'd only asked what the code does, not what it doesn't do, what would I have missed?
