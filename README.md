> **🕌 For Bakar: Word timings for 4 reciters (Yasser Al-Dosari, Maher Al-Muaiqly, Nasser Al-Qatami, Islam Sobhi) are here, 422 surah recordings in all, ready for Quran Glass to light each word. Omar Hisham's timings are made but not published, because the only per-surah copy of his recitations is an unofficial archive.org upload; they go live when you say so. Correction (25 October 2026): this page used to say some recordings leave out certain ayahs. That was wrong: a check found almost all of those ayahs are recited; they just have no word timing yet.**

# Quran recitation word timings

Word-by-word timings for full-surah recitations that Quran.com does not time. Each file has the same shape as
Quran.com's `chapter_recitations` response, so a player that follows Quran.com's timings can follow these too.
They were made for [Quran Glass](https://baklarrrr.github.io/quran-glass/).

No audio is stored here. Each timing file names the audio file it was measured on, and the player streams that
file from its source.

## Reciters

| Reciter | Audio source | Surahs | Notes |
|---|---|---|---|
| Yasser Al-Dosari | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 107 | no word timing for 15:50; 23:58; 27:75; 55:47, 51; 56:8, 40; 77:34, 45 |
| Maher Al-Muaiqly | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 105 | no word timing for 18:89; 37:175; 77:34, 45; 94:5; held back until aligned again: surah 55, 79, 91, 102 |
| Nasser Al-Qatami | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 106 | no word timing for 7:25; 36:58; 37:165; 55:16, 30, 45, 53; 77:34; 79:3; 82:17; 94:5 |
| Islam Sobhi | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 104 | no word timing for 55:36, 47; 56:39; 59:22; 77:47; 102:3; held back until aligned again: surah 3, 79 |

The ayahs in the Notes column are ones the aligner could not place, not ayahs missing from the recordings. An
independent check on 2026-10-25 ran the same recognition model over the audio around each of the 42 ayahs then
marked and read at least 70% of the ayah's letters, in order, for 38 of them (most word for word); three more were
partly read and one was inconclusive. Until they are aligned again they stay untimed: a player should show such an
ayah's text, unlit, while it is recited (between the last timed word before it and the next timed word), and never
say the reciter left it out. A surah where that stretch is too short to hold the ayah (its audio lies inside a
neighbouring word's timing) is held back until it is aligned again.

All are Hafs 'an 'Asim, murattal. [mp3quran.net](https://mp3quran.net/eng) offers a free public API for
developers but states no licence for its audio; the recordings remain the work of their reciters and publishers.
If a source replaces a file, its timings stop matching and the surah has to be aligned again.

## Files

- [`reciters.json`](reciters.json) lists the reciters. In `audioUrlTemplate`, `{ch3}` is the surah number
  padded to three digits (`036`); in `timingsUrlTemplate`, `{ch}` is the plain number (`36`). `chapters` lists
  the surahs that have timings. `partial` names surahs whose recording holds only some ayahs (`{"36": [1, 27]}`
  means ayahs 1 to 27), and `leftOut` names ayahs the aligner could not time (they are recited; see above).
- `timings/<reciter>/<surah>.json` holds one surah, for example:

```json
{"reciter": "yasser-dosari", "chapter": 112,
 "audio_file": {"audio_url": "https://cdn.mp3quran.net/audio/yasser-dosari/r1/112.mp3",
  "timestamps": [{"verse_key": "112:1", "timestamp_from": 0, "timestamp_to": 2330,
                  "segments": [[1, 0, 440], [2, 440, 800], [3, 800, 1400], ...]}, ...]}}
```

Times are milliseconds from the start of the audio file. A segment is `[word position, start, end]`, counting the
ayah's words as Quran.com does. The first ayah starts at 0, so the isti'adha and basmala before it play with it.
An ayah marked `"left_out": true` has no segments: the aligner could not time it. The name is kept so existing
players keep working, but it does not mean the recording leaves the ayah out (see Reciters); show its text, unlit,
from the end of the last timed word before it to the start of the next timed word. The audio files checked (one per reciter) are constant bitrate, so a player seeking to an ayah
lands where it should.

## How the timings were made

1. **Text.** The words of each surah come from the Quran.com API in Uthmani script. The text is used for alignment
   only and is not published here.
2. **Model.** Each recording is read by [Quran FastConformer CTC](https://huggingface.co/mrmuminov/quran-fastconformer-ctc-ar)
   (int8 ONNX), which gives a probability for each Uthmani character every 40 ms.
3. **Forced alignment.** A Viterbi search finds the most likely path through the surah's letters in order, in
   overlapping 40-second windows that are cut only between words. Letter variants are merged (أ إ آ ٱ to ا, ة to
   ه, ى ئ to ي, ؤ to و) and vowel marks count as nothing new, so spelling details never block the path. The path
   may also pass through the isti'adha and basmala, absorb audio that is not in the text (a closing du'a), go back
   and say words again (a reciter repeating an ayah or part of one; the word is timed where it was first said
   clearly), or pass over a whole ayah it cannot place, at a cost.
4. **Word edges.** The model marks a letter after it begins, so a word starts 160 ms before its first
   letter is detected, and inside an ayah each word ends where the next begins. Around the pause between ayahs a
   loudness track finds where the voice rises and falls.
5. **Checks.** A surah is published only when at least 97% of its words are placed, at most 3% of them (or one
   word) have unclear letters, no more than three unclear words come in a row, and at most 10% of its ayahs (or
   one ayah) are passed over. A passed-over ayah was meant to be accepted only when its letters were not heard in the
   audio around it, and each was checked by eye against the model's transcript; the 2026-10-25 check above showed
   that test was not reliable, so these ayahs are described as untimed, not as left out of the recording, and a
   surah is held back when an untimed ayah has no room of its own between its neighbours. When a
   recording holds only the first part of a surah, the ayah it stops after is found from the model's reading of
   its last minute and a half, and only those ayahs are timed. Such recordings are not published yet, because a
   player would show the next ayah while the rest of the file plays.

## Accuracy

Measured against Quran.com's own word timings, on seven recordings by seven reciters (3866 words). None of
these recordings was used to fit the settings; Alafasy's Al-Mulk is by a reciter whose Ya-Sin was in the fitting set
(with Husary's Al-Fatiha and Al-Ikhlas).

| Recording (Quran.com) | Words | Start error, median | 90th percentile | Reference drift per 10 min |
|---|---|---|---|---|
| AbdulBaset AbdulSamad (Murattal) 67 | 333 | 160 ms | 816 ms | -378 ms (drifts) |
| Abdur-Rahman as-Sudais 55 | 347 | 60 ms | 190 ms | -7 ms |
| Abu Bakr al-Shatri 78 | 173 | 91 ms | 931 ms | +369 ms (drifts) |
| Hani ar-Rifai 56 | 379 | 1023 ms | 1308 ms | +1259 ms (drifts) |
| Mohamed Siddiq al-Minshawi (Murattal) 36 | 724 | 60 ms | 390 ms | -22 ms |
| Sa`ud ash-Shuraym 18 | 1579 | 66 ms | 222 ms | -70 ms |
| Mishari Rashid al-`Afasy 67 | 331 | 55 ms | 185 ms | +64 ms |

On the 4 recordings whose reference matches the audio file (2981 words), word starts are
off by 62 ms at the median and 240 ms at the 90th percentile; word ends by
70 ms and 325 ms. A player lights a word at its start, so starts are what the
reader sees. Most of the end error is at the pause after an ayah, where the references disagree with each other:
Husary's mark the end where the voice stops, Alafasy's carry the last word on through the pause.

Three references drift against the audio files Quran.com serves by more than 0.3 s per 10 minutes, the limit set
after seeing these results. Hani ar-Rifai's Al-Waqi'ah also jumps by about 1.3 s after its second minute, which
points to a reference made on a different copy of the file. With those three included, starts are off by
80 ms at the median and 872 ms at the 90th percentile; for AbdulBaset and
al-Shatri, our error and the reference's cannot be separated.

## Licences and credit

- Timing data in this repository: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- Alignment model: [mrmuminov/quran-fastconformer-ctc-ar](https://huggingface.co/mrmuminov/quran-fastconformer-ctc-ar),
  CC BY 4.0, fine-tuned from NVIDIA's
  [stt_ar_fastconformer_hybrid_large_pcd_v1.0](https://huggingface.co/nvidia/stt_ar_fastconformer_hybrid_large_pcd_v1.0)
  (CC BY 4.0). Credit to NVIDIA and to the model's author.
- Quran text for alignment: the [Quran.com API](https://api-docs.quran.com/). Reference timings for measuring
  accuracy: Quran.com.
- Audio: the reciters and publishers named above. Nothing is copied here.
