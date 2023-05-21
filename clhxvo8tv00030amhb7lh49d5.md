---
title: "Present Spotify Data in Tmux with Applescript"
seoTitle: "Display Spotify Data using Tmux, Applescript"
seoDescription: "Control Spotify via AppleScript in tmux: Show song info on macOS tmux status bar, customize workflow"
datePublished: Sun May 21 2023 20:35:28 GMT+0000 (Coordinated Universal Time)
cuid: clhxvo8tv00030amhb7lh49d5
slug: present-spotify-data-in-tmux-with-applescript
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/NIo8Fd-RngE/upload/968e0d5d66834050868df98269d2c69a.jpeg
tags: spotify, scripting, tmux, scripting-languages, applescript

---

<mark>TLDR: This article demonstrates how to use AppleScript to control Spotify and display song information in a tmux status bar.</mark>

> tmux is a terminal multiplexer: it enables several terminals to be created, accessed, and controlled from a single screen. tmux may be detached from a screen and continue running in the background, then later reattached.
> 
> source: [https://github.com/tmux/tmux](https://github.com/tmux/tmux)

> **AppleScript** is a [scripting language](https://en.wikipedia.org/wiki/Scripting_language) created by [Apple Inc.](https://en.wikipedia.org/wiki/Apple_Inc.) that facilitates automated control over scriptable [Mac](https://en.wikipedia.org/wiki/Mac_(computer)) applications. The term "AppleScript" may refer to the language itself, to an individual script written in the language, or, informally, to the macOS [Open Scripting Architecture](https://en.wikipedia.org/wiki/AppleScript#Open_Scripting_Architecture) that underlies the language.[<sup>[4]</sup>](https://en.wikipedia.org/wiki/AppleScript#cite_note-Goldstein-4)[<sup>[5]</sup>](https://en.wikipedia.org/wiki/AppleScript#cite_note-Sanderson-5)
> 
> source: [https://en.wikipedia.org/wiki/AppleScript](https://en.wikipedia.org/wiki/AppleScript)

## INTRO

I have been using tmux in my workflow with [https://github.com/gpakosz/.tmux](https://github.com/gpakosz/.tmux) configuration. I've done further research into how to personalize it. Unfortunately, most plugins are written in a shell script, which is understandable as shells like **bash** are available (or built-in) in most distros or OSs. I do want to learn how to write shell scripts, not just for tmux, but as well as the universality of it.

### ***<mark>NOTE: AppleScripts will only work on a macOS ecosystem.</mark>***

Let's create a script that's **human-readable** using AppleScript to get a feel for the language and then expand on it by utilizing a shell script afterwards.

## IMPLEMENTATION: SETUP

The purpose of the following script is to explore the AppleScript language and how to utilize it to interact with the Spotify app. Additionally, we aim to display the current song information (track and artist name), as well as the player's state (song playing or paused), and whether Spotify has been launched and displays all this data in a tmux status bar.

As for most script files, the heading should be the environment that the file should be run in:

> #!/usr/bin/env osascript

1. First, we must ascertain whether Spotify is running; if it isn't, display that Spotify is currently inactive. Take note of the language's readability and the keywords employed.
    
    ```bash
    tell application "Spotify"
        if it is running then
            # 
        else
            "♫ 💤" 
        end if
    end
    ```
    
2. Now, we want to obtain and store the current track and artist's name and the player state (playing or paused) in variables, specifically, track\_name, artist\_name, and player\_state, respectively. We could also retrieve other information, such as the album\_name,
    
    ```diff
    - tell application "Spotify"
    -  if it is running then
    +   set track_name to name of current track
    +   set artist_name to artist of current track
    +   set player_state to player state as string
    -  else
    -    "♫ 💤" 
    -  end if
    - end
    ```
    
3. We now want to display the info we gathered from above. We can use an if/else statement to branch between what we display based on player state
    
    ```diff
    - tell application "Spotify"
    -  if it is running then
    -   set track_name to name of current track
    -   set artist_name to artist of current track
    -   set player_state to player state as string
    +    if player_state is equal "playing"
    +      "♫ ⏵ " & track_name & " + " & artist_name
    +    else
    +     "♫ ⏸ " & track_name & " + " & artist_name
    +    end if
    -  else
    -    "♫ 💤" 
    -  end if
    - end
    ```
    
4. And that's it. However, we can slightly refactor the code and introduce functions. Finally, let's include the maintainer's information (optional, but helpful for directing complaints when the script doesn't work as intended) and a description of the script's function for users.
    
    ```diff
    #!/usr/bin/env osascript
    
    + -- maintainer: cdrani
    + -- Returns Spotify current player state and song info
    
    + on getPlayerState(player_state)
    +     if player_state is "playing" then return "♫ ⏵ "
    +	  if player_state is "paused" then return "♫ ⏸ "
    + end getPlayerState
    
    + on printSongInfo(player_state, track_name, artist_name)
    +	getPlayerState(player_state) & track_name & " - " & + artist_name
    + end printSongInfo
    
    tell application "Spotify"
    +	if it is not running then
    +       return "♫ 💤"
    +   end
    
    	set track_name to name of current track
    	set artist_name to artist of current track
    	set player_state to player state as string
    		
    +	my printSongInfo(player_state, track_name, artist_name)
    end tell
    ```
    
5. Now, we want to make the script executable. I saved mine as "spotify.scpt" (`scpt` is the filename for AppleScript) in a scripts directory (which I am currently inside):
    
    > chmod +x ~/scripts/spotify.scpt
    
6. The final step is to integrate it into our tmux status line. I opted for the right-hand side. Inside a tmux session, enter your command prompt using `Prefix + :` and type the following based on the path the script file
    
    > set -g status-right '#(~/scripts/spotify.scpt)'
    

Great! Implementing this idea is quite simple, from conception to full realization, except for the need to close each 'tell' and 'if' statement. The song information is displayed in the top-right corner and updates upon song change, although there is a slight ~2-second delay.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684534839974/e0dc02d7-eeda-4b8e-9e53-a337853547c8.png align="center")

## AppleScript Editor

To begin, it's a good idea to use the AppleScript Editor, as it offers language features such as syntax highlighting, error compilation, quick access to language documentation, and more. Below is the aforementioned script within the editor. In the editor, we build the script to verify that there are no issues, and then we can run it, with errors and results displayed in the bottom pane.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684534122147/2267fe0a-055f-4b81-a719-8b127cad016a.png align="center")

## JavaScript Editor

AppleScript also supports JavaScript. Our script can be rewritten in JavaScript:

```javascript
// maintainer: cdrani
// Returns the current player state and song info for Spotify

function getPlayerState(playerState) {
  if (playerState === "playing") return "♫ ⏵ ";
  if (playerState === "paused") return "♫ ⏸ ";
}

function printSongInfo(playerState, trackName, artistName) {
  return getPlayerState(playerState) + trackName + " - " + artistName;
}

const spotify = Application("Spotify");

if (!spotify.running()) {
    "♫ 💤";
} else {
  const trackName = spotify.currentTrack.name();
  const artistName = spotify.currentTrack.artist();
  const playerState = spotify.playerState();

  printSongInfo(playerState, trackName, artistName);
}
```

Similar to the first script, save this one as ~/scripts/spotify.js. You can now run it in your tmux command prompt. Since this is a JS file, you need to inform AppleScript about it:

***<mark>-l language | Override the language for any plain text files. Normally, plain text files are compiled as AppleScript.</mark>***

```javascript
set -g status-right '#(osascript -l JavaScript ~/scripts/spotify.js)'
```

The integration of our scripts into tmux via the command prompt is temporary and gets cleared when the session or server is terminated. To make it permanent, we need to transfer it to our configuration file, typically saved as ~/.tmux.conf, and then source the config file to apply the changes.

```bash
echo "set -g status-right '#(~/scripts/spotify.scpt)'" >> .tmux.conf
tmux source-file ~/.tmux.conf
```

## Bonus Feature

Announcement Feature

The final feature we will incorporate enables the announcement of the current song, as well as its play/pause status, whenever the song changes.

1. Let's first introduce a new `say`, which will speak out loud our songInfo.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1684642126852/379891ca-e3b4-489b-9edc-7fadf2724429.png align="center")
    
    ```diff
    +    on sayPlayerState(player_state, track_name, artist_name)
    +        say player_state & " " & track_name & " by " & artist_name
    +    end sayPlayerState
    
    tell application "Spotify"
         -- truncated
    +    sayPlayerState(player_state, track_name, artist_name)
         my printSongInfo(player_state, track_name, artist_name)
    end tell
    ```
    
2. An issue with the above text is that the song information is repeated multiple times per song. This is likely because new data is being fed into our script every few seconds, based on the current song's state - its name, artist, whether it's playing or paused, etc. We want to limit this repetition to only occur when the song changes or the song's play/pause state changes. Let's track the song and state by defining their properties set to "missing value". jj
    
    ```diff
    -- properties to track song and state; set to non value
    +     property previous_song : missing value
    +     property previous_state : missing value
    
    --- 
    
    +    set stateExistsAndIsDifferent to previous_state is equal to missing value or previous_state is not equal to player_state
    +    set isPlayingAndSongChanged to previous_state is "playing" and previous_song is not equal to track_name
    
    +    -- update previous_state and previous_song with new states
    +    if (stateExistsAndIsDifferent or isPlayingAndSongChanged)
    +        set previous_state to player_state
    +        set previous_song to track_name
    +        sayPlayerState(player_state, track_name, artist_name)
    + 
    +        -- send command to update tmux status 
    +        do shell script "tmux set-option -g status-right '" & my printSongInfo(player_state, track_name, artist_name) & "'"
    +   end if
    ```
    
3. Finally, continuously refresh the data by running the script in a loop with a 1-second delay.
    
    ```diff
    #!/usr/bin/env osascript
    
    # maintainer: cdrani
    # Returns the current player state and song info for Spotify
    
    property previous_song : missing value
    property previous_state : missing value
    
    on getPlayerState(player_state)
        if player_state is "playing" then return "♫ ⏵ "
        if player_state is "paused" then return "♫ ⏸ "
    end getPlayerState
    
    on printSongInfo(player_state, track_name, artist_name)
        getPlayerState(player_state) & track_name & " - " & artist_name
    end printSongInfo
    
    on sayPlayerState(player_state, track_name, artist_name)
        say player_state & " " & track_name & " by " & artist_name
    end sayPlayerState
    
    + repeat
        tell application "Spotify"
            if it is not running then
                return "♫ 💤"
            end if
            
            set track_name to name of current track
            set artist_name to artist of current track
            set player_state to player state as string
        end tell
    
        set stateExistsAndIsDifferent to previous_state is equal to missing value or previous_state is not equal to player_state
        set isPlayingAndSongChanged to previous_state is "playing" and previous_song is not equal to track_name
    
        if (stateExistsAndIsDifferent or isPlayingAndSongChanged)
            set previous_state to player_state
            set previous_song to track_name
    
            sayPlayerState(player_state, track_name, artist_name)
            do shell script "tmux set-option -g status-right '" & my printSongInfo(player_state, track_name, artist_name) & "'"
         end if
     
    +    delay 1
    + end repeat
    ```
    

%[https://www.youtube.com/watch?v=v3p5hnHV1jU] 

## Conclusion

This was a simple script to showcase AppleScript and to show how it can integrate with the Apple ecosystem, here focusing on extracting Spotify info to display in a tmux status bar. In the next article, we will expand on it to add additional controls such as next, previous, pause/play, and repeat.