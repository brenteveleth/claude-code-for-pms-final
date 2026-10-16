# 06 · Sidekicks — prompts

**Context:** You joined Rook two weeks ago as PM on Dispatch.
Release 4.2 shipped on 12 August, just before you arrived, and
landed badly. You've spent five sessions on it: finding out what
went wrong, then building the fix nobody had — a working prototype,
by hand, in one sitting, for one room.

At the end of the session, type wrap up, and Claude Code saves the
prompts you wrote yourself below — not the starter prompt — updates
CLAUDE.md, and saves your work to GitHub. By Module 6 this file is a
prompt library built from your own questions.

---

### 1.

I want to save a habit as something I can reuse. When I review a brief before it goes any further, here's what I actually check for:
- It names who owns it
- It says how we'll know it worked
- The scope at the end matches the scope at the start
- It explains the problem before it proposes a fix

* Any assertions are backed with data and sources documented
* It includes a "definition of ready" for how we know we've met the requirements of the brief


Turn that into a skill called review-checklist, so I can point it at any brief and get the same check every time, without me explaining it again.

### 2.

Run review-checklist on 05-super-speed/brief.md

### 3.

push the changes we made to the brief

### 4.

run the checklist on this breif https://github.com/brenteveleth/claude-code-for-pms-final/blob/main/05-super-speed/brief.md

### 5.

now run in on this one https://github.com/kwarrenIDX/claude-code-for-pms-final/blob/main/05-super-speed/brief.md

### 6.

Schedule review-checklist to run every Monday morning, and let me know what it finds. Nothing needs to be ready for it to fire today. I'm setting the habit, not waiting on the result.
