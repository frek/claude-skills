---
name: skill-scout
description: >-
  Evaluate, critique, select, and audit AI agent skills themselves — SKILL.md
  files, skill repositories, and skill marketplaces. Use this whenever the user:
  shares a link to a skill or plugin repo (or a raw SKILL.md) and asks to
  review, critique, or assess it; is deciding whether to install a specific
  skill and wants a "is it worth it" verdict before pulling it in; asks to find
  or recommend skills for a task, stack, or recurring problem (and wants the weak
  ones filtered out); wants to audit, deduplicate, or clean up the skills they
  already have installed; or is about to install skills by tech-stack alone
  (e.g. autoskills-style auto-detection) and should sanity-check that choice.
  Triggers on phrases like "critique this skill", "should I install X",
  "find me a skill for Y", "are these skills any good", "review my installed
  skills", "покритикуй скилл", "какой скилл выбрать", "нужен ли тут скилл".
  Also fire proactively when a task is underway and a dedicated skill would
  genuinely help — flag it and offer to scout for one. Do NOT trigger for
  building ordinary (non-skill) software, generic web research unrelated to
  skills, or for using an already-loaded domain skill to do its actual job.
license: MIT
---

# skill-scout

Vet AI agent skills the way you'd vet a dependency: a skill is a chunk of prompt —
its description rides in context on every run, its body loads when it triggers. A bad
one doesn't just fail to help, it burns context and quietly tilts the output. The job
is never "install more," it's "install fewer, and only the ones that earn their weight."

**Core principle: judge from the task, not from the stack.** That React or Stripe
appears in `package.json` is not a reason to load a skill. A skill is justified only
when it fixes a *recurring mistake the agent makes without it*. Everything below
serves that test.

Respond in the user's language. Keep verdicts honest and specific — vague praise
is useless, and the point of this skill is to be the person in the room who
actually read the thing.

## Pick the mode

Route by what the user is doing. Modes share one rubric (below); each has its own
output shape.

| Mode | Trigger | What you produce |
|---|---|---|
| **Critique** | "review/critique this skill", a repo or SKILL.md link | Honest assessment: what it is, strengths, weaknesses, bottom line |
| **Decide** | "should I install X?", weighing a specific skill | A clear **worth it / skip / worth it with caveats** verdict |
| **Find** | "find/recommend a skill for Y", a task or stack | A short ranked shortlist + what to skip and why |
| **Audit** | "review my installed skills", cleanup, dedup | Keep / cut / merge table over what's already installed |
| **Scout** | proactive — a task is underway that a skill would help | One-line flag + offer to search; don't derail the task |

If the request spans modes (e.g. "find me a good animation skill and tell me if it's
worth it"), run Find then Decide on the winner.

## The rubric (all modes)

Apply these six lenses. Not every lens matters equally for every skill — weight by
what the user is deciding.

### 1. Need — does it fix a real, recurring failure?
State in one sentence the concrete mistake the agent makes *without* this skill and
how often it comes up. If you can't name the failure, that alone is a strong "skip."
"Matches my stack" is not a need.

### 2. Content quality — read the body, not the description
The description is marketing written to get the skill installed. The value is in the
`SKILL.md` body. Read it. Look for:
- **Specifics over slogans.** "Body contrast ≥ 4.5:1", "ease-out on enter",
  "line-length 65–75ch" is knowledge. "Make it beautiful and thoughtful" is filler.
- **Examples and before/after.** Shows the failure and the fix, not just a rule.
- **Honesty about limits.** Says where it does *not* apply and what it trades off.
  A skill that claims to be good at everything is good at nothing.
- **Narrow, clear scope.** One topic, covered deeply, beats a sprawling catch-all.
- **Reasoning over decrees.** "Don't do X because Y" ages better than a wall of NEVER.

### 3. Red flags
- **Filler and marketing.** Motivational prose; a mandatory scripted opening reply;
  links pushing a paid course inside the instructions.
- **Bloat.** 1000+ lines on one topic is a pile of tokens the model wades through
  every time the skill fires. Length is not quality; often the reverse.
- **Pseudo-precision.** Magic numeric "dials" (`INTENSITY: 6`) with no measurable
  scale — ritual dressed as instrument.
- **Hardcoded fashion.** Built around this year's trends; it will rot with them
  (ironically, most anti-slop skills have this problem).
- **Absolutism without context.** Wall-to-wall ALWAYS/NEVER with no "if… then."

### 4. Provenance & trust — a skill runs with your privileges
- **Author & track record.** A practitioner with real domain experience, or anon?
- **Does it run code?** Check `allowed-tools` for `Bash(...)` and any bundled
  scripts (`.mjs`, `.sh`, `.py`). Read what they do. Plain-text instructions are
  safer than a skill that shells out.
- **License.** Can it be used at work? `CC BY-NC` (non-commercial) is ambiguous for
  professional use; MIT/Apache are clean. Flag mismatches.
- **Freshness.** Last updated when? Is the repo alive?
- Note the limit of integrity checks: hashes and lock-files (autoskills-style)
  prove a file wasn't swapped in transit — they say **nothing** about whether the
  content is good or safe. A skill is a prompt; a bad prompt passes any hash. You
  still have to read it.

### 5. Context budget & overlap
- **Fewer, sharper.** Three focused skills beat ten "just in case."
- **Kill duplicates.** Two skills hitting the same need (both "anti-slop", both
  "animation") means double the context and conflicting rules — keep the better one.
- **Weigh the load.** Estimate the total lines that get pulled in. Thousands? Trim.

### 6. Test in action
Never call a skill "good" on paper alone. Recommend: take one real task the agent
used to get wrong, run it with and without the skill, compare. Visible improvement →
keep. No difference or more waffle → drop. Skills are cheap to add and cheap to
remove — say so, and encourage periodic cleanup.

## Finding skills (Find & Scout modes)

When you need to discover candidates, search — don't guess from memory, the
ecosystem moves fast. Good sources:
- **GitHub** — search `<technology> claude skill`, `<technology> SKILL.md`, or an
  author's `skills` repo. Repos are usually laid out `skills/<name>/SKILL.md`.
- **Skill directories** — skills.sh (and its leaderboard), and "awesome claude
  skills" style lists.
- **The author** — if a well-known practitioner works in the domain, check their
  repo directly; domain expertise is the best predictor of a skill worth having.

For each candidate, fetch the actual `SKILL.md` and run the rubric — a directory
listing or star count tells you nothing about content. In **Scout** mode, keep it to
a single line ("a dedicated X skill might help here — want me to look for one?") and
only dig in if the user says yes; don't hijack the task they're actually doing.

## Output shapes

**Critique** — lead with what it is in one or two sentences (size, scope, how it
works), then strengths, then weaknesses, then a one-line bottom line. Be concrete
and cite specifics from the body. Include sources when you fetched from the web.

**Decide** — open with the verdict: **worth it**, **skip**, or **worth it, with
caveats**. Then the two or three reasons that drove it. Name the single biggest
risk. If it's a "with caveats," say exactly what to watch.

**Find** — a short ranked shortlist (usually 1–3), each with a one-line why and its
main caveat, plus an explicit "what I'd skip and why." End with how to try the top
pick quickly.

**Audit** — a table of what's installed with a **keep / cut / merge** call per skill
and a one-line reason, then a short summary of what to remove to cut overlap and
context bloat. If you can read the install dir (e.g. `~/.claude/skills/`), do — read
each `SKILL.md` head rather than trusting names.

## How this differs from auto-installers

Tools like `autoskills` install by detected technology — a fine way to gather
*candidates*, but it swaps the real test (does this fix a recurring failure, is the
content good?) for a proxy (does the label appear in my deps?). Use it for a
longlist; the keep/skip call is always a human read of the actual `SKILL.md` — the
judgment this skill automates.
