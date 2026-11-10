# Automation Engineeing: Practicum Proposal

## Student Information

- **Name:**  Rafayet Hossain
- **Instructor:**  Alex Washburn
- **Date:** 10/28/2026

---

## 1. Project Title

> Provide a concise and descriptive title for your automation project.

Music Album Sorter 

---

## 2. Overview / Abstract

> Write a brief (1–2 paragraph) summary of the task you plan to automate.  
> Include the purpose of the task and why automation is beneficial.

I am automating the process of sorting and organizing my music library, for the albums that are currently on my system and for albums that I will download in the future. I often download albums through sources such as Soulseek, however every so often I will find albums that don't include proper metadata (such as album artists, song titles, album art, etc. which music players need to access), and I have to use apps that help fill out the gap. Additionally, the albums are all put on one music directory, with no regard of what artist made the album. As such, automation can help me automatically solve the metadata issues and give some proper organization to my folders.

---

## 3. Task Description

### 3.1 Manual Process Description

> Describe the task as it is currently performed manually.

1. Download an album from any user from Soulseek
2. Open the folder using MusicBrainz Picard, which inserts metadata to the album files
    - Usually the metadata is done automatically, but sometimes it will link one album to multiple different versions of the album, so I would have to fix that manually.
3. Move the folder out from my downloads to my music folder, unsorted, with just the album folder placed on the music folder directly.
4. Copy the folder to any other devices I have

Steps are performed only by me, and it occurs whenever I download an album.

### 3.2 Problem Statement

> What inefficiencies, challenges, or limitations exist in the current process?

- Since download times are slow, I have to wait until the entire album finishes downloading before I open MusicBrainz
- Moving the folder is a manual task once I do the MusicBrainz Picard step
- I would have to do this for every album

### 3.3 Automation Goal

> Clearly define what successful automation looks like.

- What will be improved? (speed, accuracy, scalability, etc.)
The speed moving the downloaded folders to the proper place in the music folder will be improved significantly, as well as the ease of use of being able to easily find any album from my folder by just going by album artist -> (album, ep, singles, ..) -> album.

---

## 4. Inputs to the Task

> Describe all inputs required for the task.

### 4.1 Input Sources

- (e.g., files, user input, web APIs, sensors, emails, etc.)

- Folders with files inside, MusicBrainz API, and possibly itunes/deezer API's for album art

### 4.2 Input Formats

- (e.g., CSV, JSON, HTML forms, plain text, images)

audio files in .flac or .mp3

### 4.3 Input Frequency

- One-time, periodic, event-driven, continuous, etc.

Event-driven - based on when I download an album.

---

## 5. Outputs and End States

> Describe the expected results of the automated process.

### 5.1 Outputs

- A properly formatted folder containing all the album tracks, sorted by album artist and the type of album it is.

### 5.2 Output Formats

- (e.g., PDF, CSV, JSON, email, SMS)

Proper folder structure with .flac or .mp3's

### 5.3 End State Conditions

> What conditions indicate the task has completed successfully?

The downloads folder is empty, and the music moved to the proper place in /music

---

## 6. Data Transformations

> Describe how data is processed and transformed throughout the task.

### 6.1 Transformation Steps

- Input cleaning - I ignore non-audio files for each folder that appears in downloads.
- Parsing / extracting - Read the existing metadata of each music file (such as the artist and album name) and check if they can be matched through the MusicBrainz API. If album art does not exist then I will then fetch it from the Deezer/Itunes API.
- Filtering / validation - I confirmed that every track has required tags, and that they match with each other. Otherwise, I would have to flag it for manual review.
- Aggregation or computation - All tracks are grouped by album, and I use the release type (album or EP or single) to determine the subfolder.
- Formatting - Destination path is `/music/(album artist)/(albums/ep/singles/..)/(album name)/(album track #.flac)`

### 6.2 Transformation Examples

> Provide at least one concrete example:

- Input → Transformation → Output

Input -> a fully downloaded album with atleast an album name and artist

```
/downloads/artist - album name/ 
    01 example.flac
    02 hi.flac
    ...
    lyrics
```

Transformation -> We ignore lyrics since it's not an audio file, read the metadata of the flac files. Some metadata is missing so we use existing metadata to search for it in MusicBrainz, which gives us an exact result, and we apply it. The cover art is not there as well, so we use Deezer to find it and embed it to each track. Then we take the finished result and check if they match across all the files inside the folder. Then we build the destination path and move the files there.

Output ->

```
/music/album artist/album/album name/
    01 example.flac
    02 hi.flac
    ...
    lyrics

/downloads/ is empty
```
---

## 7. Proposed Automation Approach

> Explain how you plan to automate the task.

### 7.1 Techniques

- (e.g., web scraping, API interaction, scripting, scheduling, event-driven automation, RPA)

Aside from API's mentioned before, I can use Python to build the automation script. I download music through the slskd client, and although I haven't read much on the documentation, I know that it could run python scripts with some event, possibly when downloads finish.

### 7.2 Tools and Technologies

- Programming languages: Python
- Libraries/frameworks: web requests for REST api's, OS pathing for building folders and moving files
- External services/APIs: MusicBrainz API, Deezer and iTunes Search APIs, slskd
- Operating system tools: Linux filesystem, slskd's event/script integration to run the script when a download finishes

### 7.3 System Architecture (Optional)

> Describe how components interact (you may include a diagram if desired).

- Download finished on slskd
- slskd runs script
- Python script reads the folder and calls MusicBrainz API
- MusicBrainz fills data
- Deezer/Itunes uses filled data to get album art
- Python script uses filled data to verify, build folder structure, and move files.

---

## 8. Pseudocode for Automated Task

> Provide high-level pseudocode that outlines the logic of your automation.
> This can look like any programming language that you are comfortable with.

```
BEGIN when downloads finish

downloads_folder = /downloads
MUSIC_DIR = "/music"

// I'm not sure if slskd can provide an exact path of the folder when it finishes downloading. If not then I can check the downloading folder to see if it's empty before continuing.
For every audio_folder in downloads_folder:
    audio_files = all .flac/.mp3 files in album_folder
    if audio_files is empty then exit with manual review

    new_tags = empty list of metadata for all files
    

    For file in audio_files:
        tags = list of metadata contained in each file
        match = Search musicbrainz using tags
        If match exists then save in new_tags
        else exit with manual review

    if any required tag (album artist, album, title, track #) is missing or
        (album artist, album) do not match across all metadata in new_tags
        then fail with manual review

    if album art is missing then: 
        search deezer/itunes api for album art
        apply album art to each metadata in new_tags

    folder_structure = MUSIC_DIR + album artist + release type + album name
    
    if folder_structure exists in the system, then exit with manual review
    create folder in system using folder_structure 

    for file in audio_files:
        modify files with new_tags
        move to folder_structure by renaming them as "# - track name"
        move non-music files from audio_folder to folder_structure

    done

```




---

## 9. Testing and Validation Plan

> How will you verify that your automation works correctly?

- Test cases:
    - An album with complete tags and album art (should be moved without needing any lookups)
    - An album with missing tags (should be filled in from MusicBrainz)
    - An album with no album art (should have art embedded from Deezer/iTunes)
    - An album that is an EP or a single (should go in the EP/single subfolder)
    - A folder with non-audio files mixed in (non-audio files should be ignored)
    - An album with tags that don't match across tracks, or one MusicBrainz can't find (should be flagged)
    - An album whose destination folder already exists (should be flagged and not overwritten)
    - Both .flac and .mp3 albums
- Expected vs. actual results: For each test case I will copy a sample album into a test downloads folder and run the script against a test music folder (not my real library). I then compare the resulting folder structure, tags, and embedded art against what I expect, and confirm the test downloads folder is empty (or still contains the album, for the flagged cases).
- Error handling strategies: If an API call fails or finds no match, the album is left untouched in downloads and flagged for manual review, and I can fall back to Picard like I do now. The script should not overwrite existing files and only moves an album after all validation has passed, so a failure cannot lose or damage music.

## 10. References (Optional)

> List any resources, documentation, or tools you plan to use.

MusicBrainz, Itunes/Deezer documentation, slskd documentation

---

## Instructor Feedback (To be completed by instructor)

- Approval Status:
- Comments:
- Suggested Improvements: