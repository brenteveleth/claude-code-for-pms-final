# 05 · Super Speed — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. You've spent three sessions finding out what went
wrong: the two piles of feedback that didn't agree, the numbers that
hid four people inside an average, and the code that settled it —
the shorter timeout is what did it.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

Helen has asked us for a brief to resolve the issues in 4.2: /Users/beveleth/Documents/GitHub/claude-code-for-pms-final/05-super-speed/director-request.txt

Reference:  rook-wiki Glossary, Team directory, and the 4.2 release comments (people are named by role; the Team directory maps roles to names). Also read the Word doc 00-rook/company/notes/handoff-from-priya.docx

create a brief for the fixes to 4.2 that we landed on in our hypothesis previously. 

the breif should include  1. who the solution is for 2.  what changes for them when its released and 3. what the fix doesn't do

focus on Fairlight, since we did a deep dive into their issues previously. ask me any questions you have before we begin

### 2.

review the context from previous sessions in CLAUDE.md and my summary (as a followup response)

### 3.

recheck it against the code

### 4.

let's work through the open issues to resolve them - one at a time

### 5.

the breif still lists "Which fade or floor, and what miss penalty?" as open

### 6.

is there a middle ground where we can institute a low-but-not-0 miss penalty so that repeat no shows do indeed go dormant, but slow responders are not severely penalized?

### 7.

with the fixes we have, would a 60s response time ever work, or is reverting to 90s mandatory?

### 8.

review the Open before we build - didn't we resolve these? if we did, can we fold them into other areas of the doc so they don't appear to be outstanding

### 9.

condense—without losing real meaning—the "the fix in order" section

### 10.

review the whole breif for similar opportunities to condense

### 11.

Save the brief exactly as it stands now as 05-super-speed/brief.md. Show me the file when it's done.

### 12.

Take the brief you just wrote and build me a working prototype, an actual screen I can click through, not a description of one. Save it as 05-super-speed/prototype.html, a single file I can just open in my browser. Show me where this would actually happen, and make at least one thing on it respond when I click it.

Follow our product's patterns and standards. Focus on concise, usable UX. Ask any questions of me before you begin

### 13.

This is a product that superheroes and their handlers use. It should feel bold, epic, and super—but still be easy to use. Focus especially on the context of the mobile experience: this is being used by a superhero, so they may be in the middle of a rescue, flying, or some other super activity. Make sure they can quickly see and respond, even while fighting crime

### 14.

Changes:

* for the prototype, let's move any and all narrative commentary about our solution outside of the actual product interfaces. the handler's dashboard and the hero's phone app should represent the way the product would look and not include our narration (like the banner at the top of the dash)
* Helen wants a clickable prototype, but she also needs to see how, over time, our fixes will impact heroes. let's break this into 3 sections that Helen can click thru: page 1, 4.2 Today. Page 2, After the fix. Each of those pages should show how a typical day for Farlight and Linda actually look: play a "fast forward" day in the life, where we see all 8 hours in about 20 seconds with a timeline at the bottom of the slide showing this play out.
* Page 3 (new page)  should be the interactive version of the post-fix product so Helen can click around the actual interface
* Handler dash needs a CTA to submit a ticket

### 15.

bring back the scoring readout on the dashboard  from the first iteration - that was useful data to show a handler and helps them understand how their hero is performing. (metrics are critical to tracking success, right?) 

change the phone app CTA to read "Crisis averted" instead of "Back to ready" when that button is clicked, give us a superhero-sized success animation before returning to the ready state

### 16.

* keep the dashboard viewport a fixed size: use a scroll area as callouts stack up, so the UI doesnt jump around
* the circular timer on mobile appears cropped left, right, top, and bottom - adjust so it isn;t clipped

### 17.

in this solution, if a routing score becomes 0, wont the hero be locked out again? explain

### 18.

what is the weighting for a pass vs a miss?

### 19.

is that the best solution to prevent our current issues with 4.2?

### 20.

yes and adjust the prototype to reflect

### 21.

commit, then confirm that our replay examples are built based on historical data

### 22.

add a fouth page, projecting how all 12 heros stats would look after 1 week and after 1 month of the fix
