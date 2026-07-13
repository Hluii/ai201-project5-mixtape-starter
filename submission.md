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


#2

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
5ef03353-4b3f-4703-b3ac-aa1f6595f894 nova
a5945dd6-3354-47d9-b673-ad05a68189e4 darius
bd377f6f-85f2-41f5-b9ab-b61d15db967f simone
36f22c80-eac1-4bdf-862b-22a85d74f80e kenji
532892af-07d5-4517-bd0a-6555fc238ba0 aaliya


0677cf83-df52-43b6-b1be-839837f7b603 Friday Energy playlist ID

Songs 
f4ece12b-ba77-4c74-90e8-50debe420c5e Frequencies