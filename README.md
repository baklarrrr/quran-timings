> **🕌 For Bakar: Word timings for 4 reciters (Yasser Al-Dosari, Maher Al-Muaiqly, Nasser Al-Qatami, Islam Sobhi) are here, 429 surah recordings in all, ready for Quran Glass to light each word. Omar Hisham's timings are made but not published, because the only per-surah copy of his recitations is an unofficial archive.org upload; they go live when you say so.**

# Quran recitation word timings

Word-by-word timings for full-surah recitations that Quran.com does not time. Each file has the same shape as
Quran.com's `chapter_recitations` response, so a player that follows Quran.com's timings can follow these too.
They were made for [Quran Glass](https://baklarrrr.github.io/quran-glass/).

No audio is stored here. Each timing file names the audio file it was measured on, and the player streams that
file from its source.

## Reciters

| Reciter | Audio source | Surahs | Notes |
|---|---|---|---|
| Yasser Al-Dosari | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 107 | the recording leaves out 15:50; 23:58; 27:75; 55:47, 51; 56:8, 40; 77:34, 45 |
| Maher Al-Muaiqly | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 109 | the recording leaves out 18:89; 37:175; 55:18, 45, 49, 51, 61, 63; 77:34, 45; 79:3; 91:4; 94:5; 102:4 |
| Nasser Al-Qatami | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 106 | the recording leaves out 7:25; 36:58; 37:165; 55:16, 30, 45, 53; 77:34; 79:3; 82:17; 94:5 |
| Islam Sobhi | [mp3quran.net (Rewayat Hafs A'n Assem - Murattal)](https://mp3quran.net/eng) | 107 | part of 54 (ayahs 1–54); the recording leaves out 3:70; 55:36, 47; 56:39; 59:22; 77:47; 79:3; 102:3 |

All are Hafs 'an 'Asim, murattal. [mp3quran.net](https://mp3quran.net/eng) offers a free public API for
developers but states no licence for its audio; the recordings remain the work of their reciters and publishers.
If a source replaces a file, its timings stop matching and the surah has to be aligned again.

## Files

- [`reciters.json`](reciters.json) lists the reciters. In `audioUrlTemplate`, `{ch3}` is the surah number
  padded to three digits (`036`); in `timingsUrlTemplate`, `{ch}` is the plain number (`36`). `chapters` lists
  the surahs that have timings. `partial` names surahs whose recording holds only some ayahs (`{"36": [1, 27]}`
  means ayahs 1 to 27), and `leftOut` names ayahs a recording skips.
- `timings/<reciter>/<surah>.json` holds one surah, for example:

```json
{"reciter": "yasser-dosari", "chapter": 112,
 "audio_file": {"audio_url": "https://cdn.mp3quran.net/audio/yasser-dosari/r1/112.mp3",
  "timestamps": [{"verse_key": "112:1", "timestamp_from": 0, "timestamp_to": 2330,
                  "segments": [[1, 0, 440], [2, 440, 800], [3, 800, 1400], ...]}, ...]}}
```

Times are milliseconds from the start of the audio file. A segment is `[word position, start, end]`, counting the
ayah's words as Quran.com does. The first ayah starts at 0, so the isti'adha and basmala before it play with it.
An ayah the recording leaves out has `"left_out": true`, no segments, and takes no time, so a player moves
straight past it. The audio files checked (one per reciter) are constant bitrate, so a player seeking to an ayah
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
   clearly), or leave out a whole ayah at a cost (some recordings skip one, as several of Ar-Rahman's refrains).
4. **Word edges.** The model marks a letter after it begins, so a word starts 160 ms before its first
   letter is detected, and inside an ayah each word ends where the next begins. Around the pause between ayahs a
   loudness track finds where the voice rises and falls.
5. **Checks.** A surah is published only when at least 97% of its words are placed, at most 3% of them (or one
   word) have unclear letters, no more than three unclear words come in a row, and at most 10% of its ayahs (or
   one ayah) are left out. A left-out ayah is accepted only when its letters are not heard in the audio around it,
   and every one published was also read back from the model's transcript of the recording. When a
   recording holds only the first part of a surah, the ayah it stops after is found from the model's reading of
   its last minute and a half, and only those ayahs are timed.

## Accuracy

Measured against Quran.com's own word timings, on seven recordings by seven reciters that played no part in
setting the method (3866 words):

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
