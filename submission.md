## Main files + their roles

    ai201-project5-mixtape-starter/
    ├── app.py                      # Flask app factory and DB setup
    ├── models.py                   # SQLAlchemy models for all entities
    ├── routes/
    │   ├── songs.py                # Song sharing, search, and rating routes
    │   ├── playlists.py            # Playlist creation and song management
    │   ├── users.py                # User profiles, streaks, notifications
    │   └── feed.py                 # Friends listening now, activity feed
    ├── services/
    │   ├── streak_service.py       # Listening streak logic
    │   ├── feed_service.py         # Friends listening now feed logic
    │   ├── search_service.py       # Song search logic
    │   ├── notification_service.py # Notification creation and retrieval
    │   └── playlist_service.py     # Playlist retrieval logic
    ├── tests/
    │   ├── test_streaks.py
    │   ├── test_search.py
    │   └── test_playlists.py
    ├── seed_data.py                # Populates DB with test data
    ├── requirements.txt
    └── .gitignore

## Data flow for at least one feature



## Any patterns you notice


## Bugs
#1


## Issue #2 — Friends Listening Now shows people from yesterday(Can reproduce)

**How I reproduced it:** Checked darius's feed via GET /feed/<id>/listening-now.
Nova appeared in the results with a listened_at timestamp roughly 3 hours old,
well past what should count as "listening now."

    curl -s http://127.0.0.1:5000/feed/ecb52e3a-a023-4e74-acdf-bdec901fd31e/listening-now | jq
    Simone 3:52:15 (Correct)
    Nove 2:07:15 (Incorrect)


**How I found the root cause:** Traced the route to get_friends_listening_now in
services/feed_service.py. The query logic itself (friend filtering, ordering,
dedup by most recent event per friend) was correct. The moment I was confident
I'd found it was spotting RECENT_THRESHOLD = timedelta(hours=24) defined at the
top of the file — a 24-hour cutoff, not a "listening now" cutoff.

**The root cause:** The cutoff used to decide whether a listening event counts as
"recent" was set to 24 hours, so anything played within the last day appeared in
the feed, including songs played the previous night. The comparison and query logic
were correct; the threshold value itself was wrong for the feature's intent.

**My fix and side-effect check:** Changed RECENT_THRESHOLD to 30 minutes, matching
the seed data's distinction between "recent" events (10-20 min old) and "older"
events (2+ hours old). After reseeding and restarting the server, darius's feed
count dropped from 3 friends (all leaking through under 24h) to 1 (only the friend
with a genuinely recent event). Confirmed friends with only older events no
longer appear.

#3

- Cannot reproduce the error with q=Anthem or q=Crown%20Heights%20Anthem

#4

## Issue #5 — The last song in a playlist never shows up(Can reproduce bug)

**How I reproduced it:** Fetched the "Friday Energy" playlist via GET /playlists/<id>/songs,
which was seeded with 7 songs. The response returned only 6.

    curl -s http://127.0.0.1:5000/playlists/0677cf83-df52-43b6-b1be-839837f7b603/songs | jq
    count : 6 
**How I found the root cause:** Traced the route to add_to_playlist and get_playlist_songs
in services/playlist_service.py and services/notification_service.py. While testing the
POST endpoint I hit an unrelated crash (missing position value on insert), which led me to
read get_playlist_songs closely. The function queries songs ordered ascending by position,
then returns `[song.to_dict() for song in songs[:-1]]` — the `[:-1]` slice was the moment
I was confident I'd found it, since it explained the exact symptom: always dropping the
last item regardless of playlist size.

**The root cause:** get_playlist_songs orders songs correctly by position, but slices the
final element off the returned list unconditionally with `songs[:-1]`. Since the list is
sorted ascending by position, the last element is always the most recently added song, so
that song was silently dropped from every response, regardless of how many songs were in
the playlist.

**My fix and side-effect check:** Removed the `[:-1]` slice so the full ordered list is
returned. Verified Friday Energy now returns all 7 songs. Also tested the boundary
condition directly: a playlist with only 1 song, which would have returned an empty list



## Thoughts
- Trying to get user ids

ID MAP
62922db6-e5dc-44b8-aae5-9a151c301d35 nova
ecb52e3a-a023-4e74-acdf-bdec901fd31e darius
95c2746a-b362-4be7-94b2-a5819bee2cf2 simone
a29c9ff4-2c22-46ae-98a4-1d906b13f0cf kenji
d7846890-ff91-4326-ab9c-169e127cafd9 aaliya


 Friday Energy playlist ID

Songs 
 Frequencies