# Clip Pipeline - Adam

Pipeline for cutting long-form videos into short clips using the **Hook + Body** structure.

## How It Works

Each topic from the video gets 2 clip variations:
- **Clip A** = Hook 1 + Body
- **Clip B** = Hook 2 + same Body

This creates 2 different openings for A/B testing.

## Technical Requirements

- **ffmpeg** installed (`brew install ffmpeg`)
- Video file (`.mov`, `.mp4`)
- Transcript with timecodes marking hooks and body segments

## Cutting Rules

### Timecode Buffers
- **5 seconds BEFORE** each segment start
- **1 second AFTER** each segment end

### Important: Re-encode, Don't Copy
Always use re-encoding instead of `-c copy`:

```bash
# CORRECT - re-encode for precise cuts
ffmpeg -ss 00:01:05 -to 00:02:30 -i input.mov \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  output.mp4

# WRONG - will cut at keyframes, eat beginning, desync audio
ffmpeg -ss 00:01:10 -to 00:02:30 -i input.mov -c copy output.mp4
```

### Why Re-encode?
- `-c copy` cuts only at keyframes (every 2-5 seconds)
- This causes the first words to be eaten
- Audio goes out of sync
- Re-encoding with buffer solves both issues

## Workflow

1. Get video file + transcript with timecodes
2. Identify Hook 1, Hook 2, and Body for each topic
3. Apply 5s/1s buffer to each segment
4. Cut and re-encode each segment
5. Concatenate: Hook 1 + Body = Clip A, Hook 2 + Body = Clip B
6. Output to `clips/` folder

## Output Naming

```
Clip_1A_TopicName_Hook1.mp4
Clip_1B_TopicName_Hook2.mp4
Clip_2A_TopicName_Hook1.mp4
Clip_2B_TopicName_Hook2.mp4
...
```

## Folder Structure

```
clip-pipeline/
├── README.md
├── clips/           # Output clips go here
├── temp/            # Temporary segments during processing
└── [video files]    # Source videos (not in repo)
```

## Notes

- Timecodes in transcripts are approximate - trust the spoken text more than exact timestamps
- Always verify the cut includes the first word of each segment
- If first words are missing, add more buffer before
