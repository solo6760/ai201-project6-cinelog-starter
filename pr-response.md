# PR Response Doc — CineLog Watchlist Feature

## AI Usage
Used AI as a devil's advocate to stress-test the architectural and design reasoning for Comments 4 and 5:
- **Questions asked:**
  - *"What counterargument would a careful code reviewer raise against defaulting public=True for a watchlist, and what privacy/behavioral tradeoff am I not acknowledging?"*
  - *"What counterargument would a reviewer raise against date-added reverse chronological sorting versus alphabetical sorting, and how can I directly engage with the maintainer's preference?"*
- **Changes made as a result:**
  - **Comment 4:** Explicitly highlighted the psychological distinction between watchlists (unfulfilled intentions) and watched collections (completed activity), acknowledging why privacy sensitivity is higher for unwatched items.
  - **Comment 5:** Addressed the "watchlist graveyard" tradeoff (older saved movies sinking out of view over time) and proposed an optional `sort_by` query parameter in future iterations to address it.

## Comment 1 — Rename
**What I did:**
- Renamed the function definition of `save_to_watchlist` to `add_to_watchlist` inside [services/watchlist_service.py](services/watchlist_service.py).
- Updated all call sites and imports, specifically in [routes/watchlist/watchlist.py](routes/watchlist/watchlist.py).
**How I verified:**
- Conducted a project-wide search for `save_to_watchlist` to confirm no remaining references existed.
- Verified that all unit tests continued to compile and run successfully.

## Comment 2 — Deduplication
**What I did:**
- Defined `AlreadyInWatchlistError` inside [services/watchlist_service.py](services/watchlist_service.py).
- Added logic in `add_to_watchlist()` to query `WatchlistEntry` and raise `AlreadyInWatchlistError` if the user already has that film in their watchlist.
- Caught `AlreadyInWatchlistError` and `FilmNotFoundError` in the watchlist route `add_film()` inside [routes/watchlist/watchlist.py](routes/watchlist/watchlist.py), returning `409` Conflict and `404` Not Found respectively.
**How I verified:**
- Referenced the duplicate checking logic of `add_to_collection()` from [services/collection_service.py](services/collection_service.py) and error handling in [routes/collection.py](routes/collection.py).
- Ran the test suite to ensure no regressions were introduced.

## Comment 3 — Missing test
**What I did:**
- Created [tests/test_watchlist.py](tests/test_watchlist.py).
- Wrote `test_add_to_watchlist_nonexistent_film_raises` to assert that trying to add a non-existent film to a user's watchlist properly raises `FilmNotFoundError`.
**How I verified:**
- Modeled the test structure directly after `test_add_to_collection_nonexistent_film_raises` in [tests/test_collection.py](tests/test_collection.py).
- Executed the tests via `pytest tests/test_watchlist.py -v` and verified that the test passes.

## Comment 4 — Default visibility
**My position:**
I recommend keeping the default watchlist visibility as `public=True`.

**Reasoning:**
CineLog is a social film-logging application designed to encourage sharing, recommendations, and movie discovery among users. Optimizing for a `public=True` default directly serves this social discovery aspect. It enables friends and other users to view watchlists without friction, making it easier to plan collaborative watch sessions, find shared interest films, or recommend movies to others. Defaulting to public maximizes user engagement and organic platform interaction, avoiding the "empty-state" social experience where most profiles appear completely private due to user inertia.

**Tradeoff acknowledged:**
The main tradeoff of a public-by-default stance is user privacy. Users who prefer to keep their upcoming watch intentions private will need to explicitly toggle their list or entries to private, creating additional friction. There is also a risk of accidental exposure if a user assumes their list is private. Defaulting to `public=False` would prioritize absolute privacy out-of-the-box, but it would significantly reduce overall social interaction across the platform as most users tend to leave default settings unchanged.

## Comment 5 — Sort order
**My position:**
I agree with the maintainer's preference and have updated `get_watchlist()` in [services/watchlist_service.py](services/watchlist_service.py) to sort by date added in descending order (`WatchlistEntry.date_added.desc()`).

**Reasoning:**
A watchlist primarily serves as an active queue of films a user intends to watch in the near future. When users view their watchlist, their immediate focus is on their most recent additions (e.g., movies added right after viewing a trailer or receiving a recommendation). Sorting by `date_added.desc()` puts these high-intent items at the top. Additionally, this aligns `get_watchlist()` with `get_collection()`, creating a consistent pattern across CineLog's service layer.

**Engagement with reviewer's point:**
The maintainer rightly points out that alphabetical sorting scatters recent additions across a large list, forcing users to search through titles rather than seeing what they recently saved. While alphabetical sorting (`Film.title.asc()`) makes finding a specific known title predictable, explicit search or filter parameters are much better suited for lookups than relying on manual list scrolling. The main tradeoff of reverse-chronological sorting is the "watchlist graveyard" effect—where older saved films sink to the bottom and get forgotten as the list grows. To resolve this long-term without compromising default queue usability, future API enhancements could accept an optional `sort_by` query parameter (allowing users to toggle between `date_added` and `title`). In the interim, defaulting to `date_added.desc()` best matches active user workflow and platform consistency.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->