# Review flow

Shared reference for Main and the planner: review classes, the held-decision category, `REVIEW.md` layout, and the review packet. Main's queue limits are in `## Human review capacity` of the workspace `AGENTS.md`.

## Choosing the review class

Every deliverable gets one class. Ask in order; the first "yes" decides.

1. Can an automated check prove the outcome (tests, measured metrics, diff of outputs)? → `machine`: no user action; one line in the final report.
2. Is the remaining question a choice that is cheap to undo and has a defensible recommendation? → `optimistic`: proceed on the default and list it under `Running on defaults` for veto.
3. Does judging it need the user's senses, taste, or a physical device (feel, look, sound)? → `experience`: it gates on the user and needs a review packet.

Use `experience` only for what automation cannot judge.

Split mixed deliverables. A new input control is `machine` for its mapping math and tests and `experience` only for how it feels in the hand; only the second part waits for the user.

## Held decisions

A decision is held in `INBOX.md`, not reviewed, when it is irreversible, costs money, installs software, pushes or publishes, changes a spec or the user's stated intent, or has no defensible default. Everything else is `optimistic`.

## `REVIEW.md` layout

    # Review

    Total: ~<N> min. Start with: <first item title>.

    ## To try

    <review packets, ordered by how much downstream work each unblocks>

    ## Running on defaults

    - `<repo>`: <choice taken> instead of <alternative>. Veto: <what reverting costs>.

When both sections are empty the file says `Nothing to review.`

## Review packet

One packet asks for one judgement. If two things need judging, write two packets.

    ### `<repo>`: <what to judge, as a question>

    - Time: ~<N> min
    - Unblocks: <work waiting on this answer>
    - Run: <one command or URL; setup steps only if unavoidable>
    - Look for: <the one or two observable things that decide the answer>
    - Answers: (a) ok  (b) <likely failure 1>  (c) <likely failure 2>  (d) other, describe
    - Recording: <path to clip, image pair, or audio render, or `none`>

Rules:

- `Run` must work as written from a fresh terminal or device; the planner or a worker has executed it.
- `Answers` lists the failure modes the planner considers most likely, so the user can reply with one letter.
- Provide a recording whenever the judgement can be made from it; then `Run` is the fallback.
- No measurements, check counts, or history in a packet; those belong in the final report.
- Write packets in the language the user uses with Main.
