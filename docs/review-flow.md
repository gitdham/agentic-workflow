# Review flow

Reference for `## Human review capacity` in the root `AGENTS.md`. Read it when writing a contract or an item in `REVIEW.md`.

## Choosing the review class

Ask in order; the first "yes" decides.

1. Can an automated check prove the outcome (tests, measured metrics, diff of outputs)? → `machine`.
2. Is the remaining question a choice that is cheap to undo and has a defensible recommendation? → `optimistic`.
3. Does judging it need the user's senses, taste, or a physical device? → `experience`.

A decision that is irreversible, costs money, installs software, pushes or publishes, or changes a spec or the user's stated intent is not a review item: it is held in `INBOX.md`.

Split mixed deliverables. "Gyro aiming" is `machine` for the direction math and tests and `experience` only for how it feels on a real phone; only the second part waits for the user.

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
