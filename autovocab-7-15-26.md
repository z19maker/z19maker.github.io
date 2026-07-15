
# Auto Vocab

## My project creates Anki flashcard set from a video. 

**Why?** I want to learn Japanese in context. Specifically I want to rewatch season 1 of blue lock but in the original language. To improve comprehension, I review the generated flash cards before watching the episode. 

**demo video.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/Jk7q8rgfjiI?si=dNcoR6e2QGSzFZXM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

**Code is available [here: https://github.com/z19maker/AutoVocab/](https://github.com/z19maker/AutoVocab)**


**How to run.** 

requires:
- uv. [How to download?](https://github.com/astral-sh/uv)
- python3. [How to download?](https://realpython.com/installing-python/)
- ffmpeg. [How to download?](https://www.wikihow.com/Install-FFmpeg-on-Windows)
- jmdict. [What is this?](https://github.com/yomidevs/jmdict-yomitan), [Download from releases page](https://github.com/yomidevs/jmdict-yomitan/releases)

```
uv venv init
source venv/bin/activate
uv pip install -r requirements.txt
```

main.py is an executable file that requires python.  
```
main.py <video-url>
```

Limitations:
- only works from Japanese to English
- only accepts mkv and mp4 format

Future improvements:
- better tokenization method
- better translation method


## Project pipeline

1. Extract audio track from .mkv 
2. Transcribe audio track
3. Transcription to vocabulary
4. Translate vocabulary
4. Vocabulary to Anki import format

## Project architecture

**AutoVocab/**

**audio.py** — Takes the input video (.mkv or .mp4), extracts the audio track using FFmpeg, and saves it to a file (.mka for mkv input, .m4a for mp4 input). Handles both MKV and MP4 containers with codec copy (no re-encoding).
- `extract_audio(mkv_file_path: Path) -> audio_file_path: Path`

**translate.py** — Loads `jmdict.json` (a Japanese-English dictionary in JSON format) at import time and builds a lookup table indexed by kanji and kana. Exposes `ja_to_en(vocab: list)` which takes a list of Japanese words and returns a dict mapping each word to its first English translation.
- `ja_to_en(vocab: list) -> dict`
- `translate(word: str) -> str` *(nested in ja_to_en)*
- `build_lookup(jmdict: dict) -> dict`

**transcribe.py** — Loads a Whisper speech-to-text model (`faster-whisper`, small, CPU, int8) at import time. Takes an audio file, transcribes the Japanese speech, and saves the full transcription to a text file.
- `transcribe(audio_file_path: Path) -> transcription_file_path: Path`

**vocabulary.py** — Takes the text transcription file and tokenizes it into individual Japanese words using `janome` (a morphological analyzer). Filters to meaningful parts of speech: nouns, verbs, adjectives, adverbs. Returns a deduplicated list of vocabulary words.
- `make_vocab(transcription_file_path: Path) -> list`
- `read_file_content(file_path: Path) -> str` *(nested in make_vocab)*

**utils.py** — Contains `check_file()` which validates that an input file exists and is a recognized video format (MKV via EBML header or MP4 via ftyp box). Contains `vocab_to_csv()` which converts a translated vocab dictionary into a CSV file (no headers, one key-value pair per row) for Anki import.
- `check_file(str_file_path: str) -> bool`
- `vocab_to_csv(vocab_dict: dict) -> csv_file_path: Path`

---

*written 7-15-26*
