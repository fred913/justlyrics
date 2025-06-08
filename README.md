# JustLyrics

A powerful TypeScript library for parsing, manipulating, and converting lyrics in multiple formats. JustLyrics provides a unified API for handling synchronized lyrics with support for syllable-level timing, multi-language annotations, and various popular lyric formats.

## Features

- **Multiple Format Support**: Convert between LRC, QRC, YRC, SRT, ALRC, LYS, LYL, LQE, SPL, and TTML formats
- **Syllable-Level Timing**: Handle precise syllable synchronization for karaoke-style applications
- **Multi-Language Support**: Built-in support for translations and pronunciation annotations
- **Voice Agent Detection**: Distinguish between main vocals, background vocals, and duet parts
- **CJK Character Detection**: Advanced handling of Chinese, Japanese, and Korean text
- **AMLL Integration**: Import and convert AMLL (Another Music Lyrics Library) format lyrics
- **Browser File API**: Read and write lyric files directly in the browser

## Installation

```bash
npm install justlyrics
```

## Quick Start

```typescript
import { CoreLyric, LyricIO, LyricFormat } from 'justlyrics';

// Create a simple line-synced lyric
const line = new CoreLyric.LineSyncedLine({
    text: "Hello world",
    start: new CoreLyric.Timestamp(1000), // 1 second
    end: new CoreLyric.Timestamp(3000)    // 3 seconds
});

// Convert to LRC format
const lrcOutput = LyricIO.Dumping.dumpLRC([line]);
console.log(lrcOutput); // [00:01.000]Hello world

// Convert to other formats
const qrcOutput = LyricIO.Dumping.dumpQRC([line]);
const srtOutput = LyricIO.Dumping.dumpSRT([line]);
```

## Core Concepts

### Timestamps

```typescript
// Create timestamps (in milliseconds)
const start = new CoreLyric.Timestamp(1500); // 1.5 seconds
console.log(start.seconds()); // 1.5
console.log(start.formatted()); // "0:01"

// Timestamp utilities
const maxTime = CoreLyric.Timestamp.max(start, new CoreLyric.Timestamp(2000));
const minTime = CoreLyric.Timestamp.min(start, new CoreLyric.Timestamp(500));
```

### Line Types

#### LineSyncedLine (Line-level timing)
```typescript
const lineSynced = new CoreLyric.LineSyncedLine({
    text: "This is a complete line",
    start: new CoreLyric.Timestamp(1000),
    end: new CoreLyric.Timestamp(4000),
    voiceAgent: {
        type: CoreLyric.VoiceAgentType.Vocal,
        n: 1
    }
});
```

#### SyllableSyncedLine (Syllable-level timing)
```typescript
const syllables = [
    {
        text: "Hel",
        start: new CoreLyric.Timestamp(1000),
        end: new CoreLyric.Timestamp(1300),
        annotations: []
    },
    {
        text: "lo",
        start: new CoreLyric.Timestamp(1300),
        end: new CoreLyric.Timestamp(1600),
        annotations: []
    }
];

const syllableSynced = new CoreLyric.SyllableSyncedLine({
    syllables,
    voiceAgent: {
        type: CoreLyric.VoiceAgentType.Vocal,
        n: 1
    },
    annotations: []
});
```

### Voice Agents

Distinguish between different vocal parts:

```typescript
// Main vocal
const mainVocal = {
    type: CoreLyric.VoiceAgentType.Vocal,
    n: 1
};

// Background vocal
const backgroundVocal = {
    type: CoreLyric.VoiceAgentType.BackgroundVocal,
    n: 1
};

// Duet (second singer)
const duetVocal = {
    type: CoreLyric.VoiceAgentType.Vocal,
    n: 2
};
```

### Annotations

Add translations and pronunciations:

```typescript
const lineWithTranslation = new CoreLyric.LineSyncedLine({
    text: "こんにちは",
    start: new CoreLyric.Timestamp(1000),
    end: new CoreLyric.Timestamp(3000),
    annotations: [
        {
            role: CoreLyric.LineAnnotationRole.Translation,
            line: CoreLyric.LineSyncedLine.fromText("Hello")
        },
        {
            role: CoreLyric.LineAnnotationRole.Prononciation,
            line: CoreLyric.LineSyncedLine.fromText("konnichiwa")
        }
    ]
});

// Extract annotations
const translations = lineWithTranslation.extractAnnotations(
    CoreLyric.LineAnnotationRole.Translation
);
```

## Format Support

### Supported Output Formats

| Format | Description                          | Extension |
| ------ | ------------------------------------ | --------- |
| LRC    | Standard LRC format                  | `.lrc`    |
| QRC    | QQ Music format with syllable timing | `.qrc`    |
| YRC    | NetEase Cloud Music format           | `.yrc`    |
| SRT    | SubRip subtitle format               | `.srt`    |
| ALRC   | Advanced LRC with JSON structure     | `.alrc`   |
| LYS    | Lyricify Syllable format             | `.lys`    |
| LYL    | Lyricify Lines format                | `.lyl`    |
| LQE    | Lyricify Quick Export                | `.lqe`    |
| SPL    | Salt Player Lyrics                   | `.lrc`    |

### Format Conversion

```typescript
const lines = [/* your lyric lines */];

// Convert to different formats
const lrcOutput = LyricIO.Dumping.dump('lrc', lines);
const qrcOutput = LyricIO.Dumping.dump('qrc', lines);
const yrcOutput = LyricIO.Dumping.dump('yrc', lines);
const srtOutput = LyricIO.Dumping.dump('srt', lines);

// Check format support
const isSupported = LyricIO.Dumping.supportDump('lrc'); // true
```

## Browser File Operations

### Reading Lyric Files

```typescript
// Request file from user
const content = await LyricFormat.requestReadLyricsFile(
    "Select lyric file",
    [".lrc", ".qrc", ".yrc"]
);

if (content) {
    // Process the lyric content
    console.log("File content:", content);
}
```

### Writing Lyric Files

```typescript
const lyricContent = LyricIO.Dumping.dumpLRC(lines);

// Save file to user's system
const saved = await LyricFormat.requestWriteLyricsFile(
    "Save lyrics as LRC",
    [".lrc"],
    lyricContent,
    { fileName: "my-lyrics.lrc" }
);

if (saved) {
    console.log("File saved successfully!");
}
```

## AMLL Integration

Convert from AMLL format:

```typescript
import { attLinesToCoreLyric, attMetaToLrcHeader } from 'justlyrics';

// Convert AMLL lines to CoreLyric format
const coreLines = attLinesToCoreLyric(amllLines);

// Generate LRC header from AMLL metadata
const lrcHeader = attMetaToLrcHeader(amllMetadata);
```

## Utilities

### CJK Character Detection

```typescript
import { isCJKChar } from 'justlyrics';

console.log(isCJKChar('你')); // true
console.log(isCJKChar('A')); // false
console.log(isCJKChar('こんにちは')); // true
```

### DOM Utilities

```typescript
import { waitFor, autoTrimContainer } from 'justlyrics';

// Wait for element to appear
const element = await waitFor<HTMLDivElement>('.lyric-container');

// Auto-trim container based on condition
autoTrimContainer(container, (el) => el.textContent?.trim() === '');
```

## Advanced Usage

### Preprocessing for Lyricify

```typescript
// Preprocess lines for better syllable splitting
const preprocessedLines = CoreLyric.Utils.preprocessLinesForLyricify(lines);
```

### Extracting Annotations

```typescript
// Extract all translations from multiple lines
const translations = CoreLyric.Utils.extractLineAnnotations(
    lines,
    CoreLyric.LineAnnotationRole.Translation
);

// Extract all pronunciations
const pronunciations = CoreLyric.Utils.extractLineAnnotations(
    lines,
    CoreLyric.LineAnnotationRole.Prononciation
);
```

### Word Syllable Splitting

```typescript
// Split a syllable into word components
const syllable = {
    text: "Hello world",
    start: new CoreLyric.Timestamp(1000),
    end: new CoreLyric.Timestamp(3000),
    annotations: []
};

const splitSyllables = CoreLyric.Utils.splitWordSyllables(syllable);
```

## API Reference

### CoreLyric Namespace

- `Timestamp` - Time representation with utilities
- `LineSyncedLine` - Line-level synchronized lyrics
- `SyllableSyncedLine` - Syllable-level synchronized lyrics
- `VoiceAgentType` - Enum for vocal types
- `LineAnnotationRole` - Enum for annotation types
- `Utils` - Utility functions for lyric processing

### LyricIO Namespace

- `Dumping` - Functions for converting to various formats
- `Abstraction.ALRC` - ALRC format type definitions

### LyricFormat Namespace

- `TYPES` - Supported format definitions
- `requestReadLyricsFile()` - Browser file reading
- `requestWriteLyricsFile()` - Browser file writing

## Contributing

We welcome contributions from the community. Please read our license terms to understand the different rights for Tier 1 and Tier 2 licensees.

## Support

For support and questions, please check our documentation or open an issue on the project repository.

## License

This project is licensed under the Lakelink's University Campus Access Public License (LKLK-CAPL) Version 3.0.


© [2025] Sheng Fan

Licensed under the Lakelink's University Campus Access Public License (LKLK-CAPL) Version 3.0:
https://github.com/Lakelink/LKLK-CAPL/blob/main/LICENSE

By exercising your rights under LKLK-CAPL, you affirm that you have read and understand this license and you confirm whether you're authorized as Tier 1 or Tier 2. Rights might differ depending on your authorization status.