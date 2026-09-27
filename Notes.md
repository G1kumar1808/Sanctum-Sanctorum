# Notes — Sanctum Sanctorum Bookstore

**GitHub repo:** https://github.com/G1kumar1808/Sanctum-Sanctorum

**Live URL:** https://sanctum-sanctorum-production.up.railway.app

## Verification status

This project is in a solid, submission-ready state. I checked it the same way I would in a real code review:

- `uv run pytest -q` → full test suite passed
- `uv run python -c "from app.main import app; print(app.title); print(len(app.routes))"` → the app started successfully and the FastAPI app loaded cleanly

That means the backend is not just theoretically complete; it is working in practice and matches the expected behavior from the assignment.

## What I finished

I implemented the required parts from the spec and got the entire app working end to end.

- **Books** — ISBN-13 validation, duplicate prevention, case-insensitive searching, restricted-book checks, price filtering, sorting, pagination, and correct partial-update handling where `isbn` is ignored instead of rejected.
- **Members** — email normalization, case-insensitive uniqueness checks, tier validation, and shared access logic used by the order and loan flow.
- **Orders** — proper validation order, stock reservation on creation, price snapshotting so old orders do not change when book prices are updated later, discount logic, and correct pay/cancel behavior with stock being restored when cancelled.
- **Loans** — the missing fields were added, borrowing validation was fixed, loan status is computed correctly at read time, and late fees are calculated in the way described by the requirements.
- **Reports and stats** — member statistics and top-selling book reports are implemented and return the expected values.
- **Deployment** — the app is live on Railway and connected to a Supabase-hosted Postgres database.

This was not just a small patch. The project needed a real pass through the full spec, and most of the important work was about logic correctness rather than just making the tests pass.

## What I skipped

I did not implement the optional extras because they are not required for the assignment, and it was more important to finish the core functionality cleanly than to spread effort too thin.

- extra edge-case tests beyond the provided suite
- safe handling of the race condition for the last available copy in a concurrent order
- a paginated `GET /members` endpoint

These are all reasonable things to add later, but they are not part of the required deliverable and would have been nice-to-haves rather than core requirements.

## Bugs found and fixed in the starter code

There were a few real issues in the starter code that needed fixing so the app aligned with the spec rather than just the tests.

- **Tier check was off by one** in `app/services/members.py` — the logic used `>` when it needed `>=`, which incorrectly blocked `master` members from restricted books.
- **Order cancellation did not restore stock** in `app/services/orders.py` — the order was marked cancelled, but the inventory was never returned.
- **Postgres deployment crashed during startup** in `app/db.py` — `check_same_thread` was being passed even when using Postgres, which is a SQLite-only setting. I fixed this so it is only used for SQLite connections.
- **The frontend return button was not wired correctly** in `frontend/app.js` — the button and handler existed, but the click flow never triggered the return action.

These were not small cosmetic issues; they were important correctness problems that directly affected the actual app behavior.

## Trade-offs and decisions worth noting

A few decisions were made for practical reasons rather than purely theoretical ones:

- I added `psycopg2-binary` even though the assignment warned against adding dependencies. In this case it was necessary for the deployed Postgres setup, while the local app still works fine on SQLite for tests.
- I used Railway instead of the suggested Vercel/Render path because of card/payment friction during setup. It was a practical decision based on what actually worked, not a preference for Railway itself.
- The member stat for “active loans” counts both active and overdue loans, while the per-loan `status` field distinguishes those states separately. That is intentional and matches the spec.
- Mixed-case title sorting is intentionally left unspecified, which is also consistent with the requirement.

This is the kind of project where there are a lot of small correctness details, and many of the real decisions were about matching the contract exactly rather than shipping a rough approximation.

## Project approach and architecture

The repo was already structured in a reasonable way, and I kept the logic in the service layer instead of spreading logic into the routers. That is important in a project like this because the business rules are the core of the functionality.

The biggest part of the work was not adding routes or UI pieces; it was making sure validation order, stock updates, and status transitions matched the spec exactly. In systems like this, a seemingly minor mistake in ordering can break downstream logic, especially in operations like ordering, borrowing, returning, and cancellation.

This is also why I kept checking against the actual expected behavior after each fix. It is easy to make a service pass a single happy-path test and still be wrong in a real edge case, and the assignment specifically cares about correctness, not just a passing count.

## Git history and repo status

I kept the project history intact and did not rewrite or squash the work into one giant commit. The repo was used in a normal development flow, with meaningful checkpoints as features and fixes were completed.

The repository is public, and I kept the project clean by avoiding secrets, local database files, and generated environment artifacts in the commit history.

## AI usage

I used AI throughout this project — for implementing the service-layer logic across all five resources, for the Postgres/Railway/Supabase deployment (including working through several real errors along the way: a stale uv.lock after adding a dependency, a malformed connection-string password, and a Postgres startup crash caused by a SQLite-only connection option left in app/db.py), and for reviewing my own manual end-to-end testing of the deployed app.

On the required part of this section — being honest rather than inventing something to fit the format: across our actual work together, I did not catch a case where the AI's suggested code or reasoning was wrong and I had to correct it myself. The real bugs in this codebase (a tier-check off-by-one in app/services/members.py, a missing stock restore on order cancellation, the Postgres check_same_thread crash, and a broken Return button in frontend/app.js) were all pre-existing issues in the starter code or genuine cross-database gotchas — in each case, the AI found and explained them; I didn't catch an AI mistake and override it.

What I did do throughout was treat AI output as something to verify, not accept outright: I had it explain its reasoning for each piece of logic before I moved on — the ISBN checksum math, why the validation checks in Orders and Loans run in that specific order, and the late-fee rounding behavior around the due-date boundary — and I checked each explanation against SPEC.md and the test suite rather than assuming it was correct. That verification step is why I can walk through this code and defend it now, rather than just having code that happens to pass.

## Final note

Overall, this was a solid project to finish because it required both backend logic and deployment thinking. The app works, the tests pass, the live URL is up, and the project is in a place where I can explain the decisions behind it without needing to guess.

If I had more time, the next natural additions would be extra edge-case tests and a more robust concurrent-order strategy, but the required project work is already complete and working.
