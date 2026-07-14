# PR Response Doc — CineLog Watchlist Feature

## AI Usage

<!-- Fill in at the end. Explain how AI was used for codebase orientation,
debugging support, and commit-message verification. Also explain how all
suggestions were checked against the actual CineLog codebase. -->

## Comment 1 — Rename

**What I did:**

I renamed `save_to_watchlist()` to `add_to_watchlist()` in
`services/watchlist_service.py` and updated the import and function call in
the watchlist route.

**Why:**

The new name follows CineLog's existing `verb_to_noun` naming convention and
matches the related `add_to_collection()` service function. This makes the
watchlist and collection APIs more consistent and easier to understand.

**How I verified:**

I searched the repository for remaining references to
`save_to_watchlist()` and confirmed that none remained. I then ran the
existing pytest suite and confirmed that all tests passed.

## Comment 2 — Deduplication

**What I did:**

I added an `AlreadyInWatchlistError` exception and added a query inside
`add_to_watchlist()` that checks for an existing `WatchlistEntry` with the
same `user_id` and `film_id`. If one exists, the service raises the exception
instead of creating another row.

**Why:**

I followed the same pattern used by `add_to_collection()` in
`services/collection_service.py`. Checking before insertion gives callers a
clear domain-specific error and prevents duplicate watchlist entries instead
of relying only on a database failure.

**How I verified:**

I compared the implementation with the existing collection-service
deduplication logic, ran the full test suite with `pytest tests/ -v`, and
confirmed the existing tests still passed.

## Comment 3 — Missing test

**What I did:**

I created `tests/test_watchlist.py` and added
`test_add_to_watchlist_nonexistent_film_raises()`. The test creates an
isolated in-memory database and a sample user, then calls
`add_to_watchlist()` with a film ID that is not present.

**Why:**

I modeled the test after
`test_add_to_collection_nonexistent_film_raises()` in
`tests/test_collection.py`. This verifies that the watchlist service raises
the expected `FilmNotFoundError` instead of allowing a database integrity
error or creating an invalid entry.

**How I verified:**

I ran `pytest tests/test_watchlist.py -v` to confirm the new test passed, then
ran `pytest tests/ -v` to confirm the full suite still passed.

## Comment 4 — Default visibility

**My position:**

**Reasoning:**

**Tradeoff acknowledged:**

## Comment 5 — Sort order

**My position:**

**Reasoning:**

**Engagement with reviewer's point:**

## Comment 6 — Rebase

**What conflicted:**

**How I resolved it:**

**How I verified no conflict remains:**

## PR Description

### Feature overview

### Design decisions

### Testing performed

### Manual testing steps