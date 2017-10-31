---
layout: post
title: Making an Automated Spotify Playlist
---

I got to mess around with the Spotify API recently. It was a ton of fun, and my brain went in one direction after using it for a few hours: automating playlists.

Now, I know that Spotify already has an incredibly robust recommendation engine. There's been <a href="http://blog.galvanize.com/spotify-discover-weekly-data-science/" target="_blank">a ton</a> written about it. But as a DJ, and an arrogant one at that, I know it still has a ways to go.

Sure, I can get an endless list of recommended songs based off of an artist, or a playlist, or even a single song. But what if I don't know what I really want? What if I want to throw myself, or someone else, a bit of a curveball? What if I want to hop from Justin Timberlake to Depeche Mode to Outkast to Aphex Twin? I thrive in a bit of chaos, and Spotify's algorithims can seem a little too tidy sometimes.

One approach I want to try taking is making playlists based off BPM (tempo). This is what I used to do when I DJ'd: get set in a tempo and then bounce around a bunch of different genres that all had similar speeds. It was a fun way to play different types of music, and people always appreciated a juxtaposition that had a clean transition.

Although I can't quite access every song in Spotify's database yet, I can get a decently-sized library by accessing my playlists. I've been using Spotify for six years now, and in creating my own playlists and saving playlists created by others, I have 14,000+ songs I can access that span a range of different genres.

First I'll install my necessary packages to get going. I'll be using the Spotipy client in Python.

```
import spotipy
import sqlite3
from spotipy.oauth2 import SpotifyClientCredentials
import spotipy.util as util
from private_files import client_id, client_secret, redirect_uri
import numpy as np
import pandas as pd
from time import sleep
token = util.prompt_for_user_token('partymuzik',scope='playlist-modify-public',client_id=client_id,client_secret=client_secret,redirect_uri=redirect_uri)
spotify = spotipy.Spotify(auth=token)
```

Next I'll iterate through every playlist I have (nearly 350!) and save their IDs.

```
def create_list_of_playlists(spotify):
    all_uris = []
    owners = []
    i = 0
    while True:
        playlists = spotify.user_playlists('partymuzik', limit=50, offset=i)
        if len(playlists['items']) > 0:
            my_list = playlists['items']
            for things in my_list:
                all_uris.append(things['uri'])
                owners.append(things['owner']['id'])
            i += 50
        else:
            break
    return all_uris, owners
```

Then I'll iterate through these playlist IDs to get a list of unique track IDs.

```
def create_list_of_tracks(all_uris, owners):
    track_uris = []
    for i in range(len(all_uris[:20])):
        tracks = spotify.user_playlist_tracks(owners[i], all_uris[i])['items']
        for j in range(len(tracks)):
            print(i, j)
            sleep(0.1)
            try:
                track_id = tracks[j]['track']['id']
                if track_id != None:
                    track_uris.append(track_id)
            except:
                track_id = 'None'
    return list(set(track_uris))
```

Then I'll go through these track IDs and get relevant information for the track, such as its artist, name and popularity.

```
def get_track_details(track):
    track = spotify.track(track)
    l = []
    requests = [track['name'], track['artists'][0]['name'], track['artists'][0]['uri'], 
               track['popularity'], track['uri'], track['album']['uri']]
    for things in requests:
        try:
            l.append(things)
        except:
            l.append('None')
    return l
def get_all_track_details(track_uris):
    all_tracks = []
    for i in range(len(track_uris)):
        print(i)
        sleep(0.1)
        all_tracks.append(get_track_details(track_uris[i]))
    return(all_tracks)
```

I'll iterate again and get musical information from each track, including tempo (this is a separate request in their API)

```
def get_audio_details(row):
    try:
        audio_features = spotify.audio_features(row)[0]
    except:
        audio_features = ''
    l = []
    my_list = ['acousticness', 'danceability', 'energy', 'instrumentalness', 'key', 'liveness', 'loudness', 'mode', 'speechiness', 'tempo', 'time_signature', 'valence']
    for things in my_list:
        try:
            l.append(audio_features[things])
        except:
            l.append('None')

    return l
def get_all_audio_details(track_uris):
    all_tracks = []
    for i in range(len(track_uris)):
        print(i)
        sleep(0.1)
        all_tracks.append(get_audio_details(track_uris[i]))
    return(all_tracks)
audio_details = get_all_audio_details(track_uris)
```

Now I can create a dataframe that has every track in every playlist I follow, along with each track's popularity and tempo (along with a bunch of other musical features).

<img src="/../images/playlist_df_head.png" width="800" />


Now comes the fun part! I can make a small function where I choose an initial tempo and a 'popularity percentile' (i.e. I want this song to be in the top 80th percentile as far as popularity). It will then pick a song in that tempo range and then randomly walk to a new tempo that's close, picking a song with a nearby tempo that fits these parameters. When it makes the number of songs I want, it'll make me a new Spotify playlist with those songs!

```
def make_steps(start_tempo, number_of_steps, new_df, percentile, spotify):
    i = 0
    tempo = start_tempo
    track_uris = []
    while i < number_of_steps:
        tempo, track_uri = pick_song(tempo, track_uris, new_df, percentile)
        track_uris.append(track_uri)
        i += 1
    playlist = spotify.user_playlist_create('partymuzik', 'Test Playlist One')
    spotify.user_playlist_add_tracks('partymuzik', playlist['id'], track_uris, position=None)

def pick_song(tempo, track_uris, new_df, percentile):
    test = new_df[((new_df['tempo'] > tempo - 2) & (new_df['tempo'] < tempo + 2)) | ((new_df['tempo'] > (tempo*2) - 4) & (new_df['tempo'] < (tempo*2) + 4)) | ((new_df['tempo'] > (tempo/2) - 1) & (new_df['tempo'] < (tempo/2) + 1))]
    test = test[~test.track_uri.isin(track_uris)]
    popularity_minimum = np.percentile(test['track_popularity'], percentile)
    test = test[test['track_popularity'] > popularity_minimum].reset_index(drop=True)
    number = random.choice(list(test.index))
    test = test.iloc[number, :]
    print(test['artist'], test['song'], test['tempo'], test['valence'])
    return test['tempo'], test['track_uri']

````

Here's a test playlist created at a tempo of 100. The playlist has a nice balance of hip-hop, R&B, classic rock and soul and Chumbawamba. 

<img src="/../images/sample_playlist.png" width="800" />


If I were DJing, this would be a reasonable start towards an uptemo playlist that could blend well together. Maybe some songs are too slow (Chris Stapleton, Otis Redding). I can look into release dates and filter by the year released, as well as other components of Spotify's audio features to see how I can adjust!

In the meantime, enjoy the playlist: 

<iframe src="https://open.spotify.com/embed/user/partymuzik/playlist/25JayByKdP9DrShqiOXr7W" width="300" height="380" frameborder="0" allowtransparency="true"></iframe>

