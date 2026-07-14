# PR Response Doc — CineLog Watchlist Feature

## AI Usage

I used AI tools during codebase orientation to compare the watchlist service
with the existing collection-service patterns. In particular, I used AI to
help explain how `add_to_collection()` validates film existence, checks for
duplicates, and raises domain-specific exceptions. I verified that explanation
against the actual implementation before writing the watchlist version.

I also used AI as a review aid for Comments 4 and 5. I asked for potential
counterarguments to keeping watchlists public by default and sorting them
alphabetically. I revised my responses to acknowledge the privacy risk of a
public default and the discoverability advantage of date-added ordering, while
keeping my final decisions grounded in CineLog's community and browsing use
cases.

Finally, I used AI to check whether my Git commit messages followed
conventional commit format. I verified the suggestions myself and used
interactive rebase to ensure each commit represented one logical change.

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

I chose to keep `public=True` as the default for watchlist entries.

**Reasoning:**

CineLog is designed as a community film-tracking application, so sharing film
interests is part of the platform's broader purpose. A public watchlist can
support future social features such as recommendations, profile discovery,
and conversations between users with similar interests.

Keeping the default public also reduces friction for users who join CineLog
because they want to participate in its community features. They can add a
film to their watchlist without needing to configure visibility each time.

**Tradeoff acknowledged:**

The main disadvantage is that some users may assume a personal watchlist is
private. A private default would better protect users who do not intend to
share their viewing interests.

To address that concern, CineLog should clearly communicate visibility in the
interface and eventually allow users to choose public or private when adding
an entry. For the current community-focused design, I am keeping the public
default, but I recognize that this decision should be revisited when user
accounts and privacy controls become more developed.

## Comment 5 — Sort order

**My position:**

I chose to keep the watchlist sorted alphabetically by film title.

**Reasoning:**

I understand the maintainer's point that recently added films may represent
what a user is currently most interested in. However, I see a watchlist
primarily as a reference list that users browse when deciding what to watch.

Alphabetical ordering gives the list a stable and predictable structure. As
the watchlist grows, a user can locate a known title without needing to
remember when it was added. This differs from the collection feature, which
acts more like a viewing history and therefore benefits from newest-first
ordering.

**Engagement with reviewer's point:**

Date-added ordering would make recently discovered films easier to revisit,
and it may be preferable for users who treat the watchlist like an activity
feed. The limitation of alphabetical ordering is that newly added films do
not receive priority.

For CineLog's current implementation, I believe predictable lookup is more
useful than recency. A future version could support a query parameter or user
preference for alphabetical and newest-first sorting.

## Comment 6 — Rebase

**What conflicted:**

The watchlist feature branch was based on an older version of the project that
used integer film IDs, while the updated main branch had migrated film IDs to
UUIDs.

**How I resolved it:**

I rebased my feature branch onto the updated main branch and updated the
watchlist code to use UUIDs consistently. I also updated the remaining
documentation and comments that still referred to integer film IDs.

**How I verified no conflict remains:**

I completed the rebase successfully, ran the full test suite with
`pytest tests/ -v`, and confirmed the branch history contains no merge
commits.

## AI Usage

I used AI tools during codebase orientation to compare the watchlist service
with the existing collection-service patterns. In particular, I used AI to
help explain how `add_to_collection()` validates film existence, checks for
duplicates, and raises domain-specific exceptions. I verified that explanation
against the actual implementation before writing the watchlist version.

I also used AI as a review aid for Comments 4 and 5. I asked for potential
counterarguments to keeping watchlists public by default and sorting them
alphabetically. I revised my responses to acknowledge the privacy risk of a
public default and the discoverability advantage of date-added ordering, while
keeping my final decisions grounded in CineLog's community and browsing use
cases.

Finally, I used AI to check whether my Git commit messages followed
conventional commit format. I verified the suggestions myself and used
interactive rebase to ensure each commit represented one logical change.
6. Final PR description

Place this at the bottom of pr-response.md, and use the same content in GitHub:

## PR Description

### Feature overview

This pull request adds a watchlist feature to CineLog. Users can add films
they want to watch later and retrieve their saved watchlist through the
watchlist service and REST endpoints.

The implementation includes:

- A `WatchlistEntry` model connected to users and films
- An `add_to_watchlist()` service function
- Duplicate-entry prevention
- Validation for nonexistent film IDs
- A GET endpoint for retrieving a user's watchlist
- UUID-compatible film references after rebasing onto the updated `main`

### Design decisions

I kept watchlist entries public by default because CineLog is a
community-oriented film platform and public lists can support discovery and
future social features. I acknowledge that a private default would provide
stronger privacy protection, so visibility should be clearly communicated and
made configurable in a future interface.

I kept watchlists sorted alphabetically by film title. I chose this because a
watchlist functions primarily as a browse-and-lookup list, and alphabetical
ordering remains predictable as the list grows. Date-added ordering would
better highlight recent interest, so supporting multiple sort options would be
a useful future improvement.

### Manual testing steps

## Manual Testing Steps

1. Activate the virtual environment:

   ```bash
   source .venv/Scripts/activate
   ```

2. Install the project dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Run the automated test suite to verify all tests pass:

   ```bash
   pytest tests/ -v
   ```

4. Start the Flask application:

   ```bash
   FLASK_APP=app:create_app flask run
   ```

5. Open a second terminal and create a sample user and film if the database is empty.

6. Add a film to a user's watchlist by sending a POST request to:

   ```
   POST /watchlist/<user_id>/add
   ```

   Include the film UUID in the request body.

   **Expected result:** A `201 Created` response containing the newly created watchlist entry.

7. Attempt to add the same film to the user's watchlist again.

   **Expected result:** The service prevents duplicate entries and returns the appropriate error instead of creating a second watchlist entry.

8. Attempt to add a film using a UUID that does not exist.

   **Expected result:** The service raises `FilmNotFoundError` and does not create a watchlist entry.

9. Retrieve the user's watchlist:

   ```
   GET /watchlist/<user_id>
   ```

   **Expected result:** The watchlist contains the expected film information and metadata.

10. Verify that all automated tests continue to pass after the changes:

    ```bash
    pytest tests/ -v
    ```