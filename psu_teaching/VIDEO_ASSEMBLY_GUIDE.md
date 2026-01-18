# Video Assembly Guide - Video-Based SRT Workflow

**Purpose:** Assemble lecture videos with perfectly synchronized slide transitions using video-based SRT transcripts.

**Status:** ✅ VERIFIED - Used successfully for Week 0 Course Overview video

---

## The Problem We Solved

### Original Approach (BROKEN)
1. Generate audio from script → MP3 file
2. Generate SRT transcript from MP3
3. Extract slide markers from SRT
4. Assemble video with marker timestamps

**Why it failed:** ElevenLabs processes SSML `<break time="X.Xs" />` tags inconsistently, causing progressive timing drift (11-89 seconds) between SRT timestamps and actual video audio.

### Corrected Approach (WORKING)
1. Generate audio from script → MP3 file
2. Assemble TEMP video with equal time distribution
3. **Generate SRT from TEMP video** (not from MP3!)
4. Extract slide markers from video-based SRT
5. Reassemble FINAL video with corrected marker timestamps

**Why it works:** SRT generated from video matches the actual audio timing perfectly, eliminating all drift.

---

## Prerequisites

### Required Tools
- **ffmpeg** - Video/audio processing (`brew install ffmpeg`)
- **decktape** - HTML slides to PDF (`npm install -g decktape`)
- **uv** - Python package manager (already installed)
- **transcribe script** - `~/opt/scripts/transcribe` (Gemini-based transcription)

### Required Files
- Script with slide markers: `scripts/week0_commitment/0.X_lesson.md`
- Audio file: `audio/week0_commitment/0.X_lesson.mp3`
- Slides HTML: `slides/week0_commitment/0.X_lesson.html`

### Verify Prerequisites
```bash
# Check tools
which ffmpeg ffprobe decktape
ls -la ~/opt/scripts/transcribe

# Check required files exist
ls -la workshops/climate_policy/scripts/week0_commitment/0.X_*.md
ls -la workshops/climate_policy/audio/week0_commitment/0.X_*.mp3
ls -la workshops/climate_policy/slides/week0_commitment/0.X_*.html
```

---

## Complete Workflow - Step by Step

### Phase 1: Initial Assembly (Rough Timing)

**Purpose:** Create a temporary video to extract accurate audio timing

```bash
cd /Users/tanwa/gt/psu_teaching/crew/video_producer
cd workshops/climate_policy

# Assemble initial video with equal time distribution
uv run python scripts/assemble_video.py \
  --lesson 0.5_course_overview_expectations \
  --week week0_commitment \
  --type TEMP
```

**Output:** `videos/week0_commitment/0.5_course_overview_expectations_TEMP.mp4`

**What this does:**
- Converts HTML slides to PDF
- Extracts slide images (PNG files)
- Distributes audio equally across all slides
- Creates rough video for timing extraction

**Expected duration:** ~5-10 minutes

**Verification:**
```bash
# Check video was created
ls -lh videos/week0_commitment/0.5_*_TEMP.mp4

# Check duration matches audio
ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 \
  videos/week0_commitment/0.5_*_TEMP.mp4
```

---

### Phase 2: Generate Video-Based SRT

**Purpose:** Extract transcript with timestamps that match actual video audio

```bash
# Generate SRT from the TEMP video (NOT from source audio!)
~/opt/scripts/transcribe \
  videos/week0_commitment/0.5_course_overview_expectations_TEMP.mp4 \
  --prompt "Please provide output in SRT subtitle format with timestamps" \
  -o transcripts/week0_commitment/0.5_VIDEO.srt
```

**Output:** `transcripts/week0_commitment/0.5_VIDEO.srt`

**What this does:**
- Extracts audio from video file
- Sends to Gemini API for transcription
- Returns SRT format with precise timestamps
- Timestamps match actual video audio (not source MP3)

**Expected duration:** ~2-5 minutes

**API Cost:** ~$0.10 per 16-minute video

**Verification:**
```bash
# Check SRT was created
ls -lh transcripts/week0_commitment/0.5_VIDEO.srt

# Spot check timestamps
head -30 transcripts/week0_commitment/0.5_VIDEO.srt

# Verify format
# Should see:
# 1
# 00:00:00,750 --> 00:00:07,320
# Welcome to the final lesson...
```

---

### Phase 3: Extract Slide Markers

**Purpose:** Get slide transition timestamps aligned to actual video audio

```bash
# Extract markers using video-based SRT
uv run python scripts/extract_markers_from_script.py \
  scripts/week0_commitment/0.5_course_overview_expectations.md \
  transcripts/week0_commitment/0.5_VIDEO.srt
```

**Output:** `transcripts/week0_commitment/0.5_VIDEO_markers.json`

**What this does:**
- Parses script to find `<!-- SLIDE: Title -->` markers
- Parses video-based SRT to get text with timestamps
- Matches marker positions to SRT segments
- Outputs JSON with slide timestamps

**Expected duration:** ~10-30 seconds

**Verification:**
```bash
# Check markers JSON was created
cat transcripts/week0_commitment/0.5_VIDEO_markers.json | jq

# Verify slide count matches
echo "Script markers:"
grep -c "<!-- SLIDE:" scripts/week0_commitment/0.5_*.md
echo "Extracted markers:"
cat transcripts/week0_commitment/0.5_VIDEO_markers.json | jq '. | length'

# Spot check timing
cat transcripts/week0_commitment/0.5_VIDEO_markers.json | jq '.[0:5]'
```

**Expected output:**
```json
[
  {
    "slide_index": 0,
    "title": "Course Overview and Expectations",
    "timestamp": 0.75
  },
  {
    "slide_index": 1,
    "title": "Learning Objectives",
    "timestamp": 37.24
  },
  ...
]
```

---

### Phase 4: Final Assembly with Corrected Markers

**Purpose:** Reassemble video with perfectly aligned slide transitions

**Option A: Use Custom Assembly Script (Recommended)**

```bash
# Create assembly script for this video
# (You can use assemble_0.5_with_markers.py as template)

uv run python scripts/assemble_video_with_markers.py \
  --lesson 0.5_course_overview_expectations \
  --markers transcripts/week0_commitment/0.5_VIDEO_markers.json \
  --output videos/week0_commitment/0.5_course_overview_expectations_FINAL.mp4
```

**Option B: Modify Existing Script**

Edit `scripts/assemble_video.py` to accept `--markers` parameter, or create a custom script like this:

```python
#!/usr/bin/env python3
import json
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent))
from assemble_video import VideoAssembler

def main():
    workshop_root = Path(__file__).parent.parent
    lesson_id = "0.5_course_overview_expectations"

    # Load video-based markers
    marker_json = workshop_root / f"transcripts/week0_commitment/{lesson_id}_VIDEO_markers.json"
    with open(marker_json, 'r') as f:
        markers = json.load(f)
    timestamps = [m['timestamp'] for m in markers]

    # Setup paths
    audio_file = workshop_root / f"audio/week0_commitment/{lesson_id}.mp3"
    output_file = workshop_root / f"videos/week0_commitment/{lesson_id}_FINAL.mp4"
    images_dir = workshop_root / f".temp_video_assembly/{lesson_id}_TEMP/images"

    # Assemble
    assembler = VideoAssembler(workshop_root)
    assembler.create_video_from_images(
        images_dir, audio_file, output_file, "FINAL", timestamps
    )

if __name__ == "__main__":
    main()
```

**Output:** `videos/week0_commitment/0.5_course_overview_expectations_FINAL.mp4`

**Expected duration:** ~5-10 minutes

**Verification:**
```bash
# Check final video was created
ls -lh videos/week0_commitment/0.5_*_FINAL.mp4

# Verify duration
ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 \
  videos/week0_commitment/0.5_*_FINAL.mp4

# Should match source audio duration (±2 seconds)
```

---

### Phase 5: Quality Verification

**Manual Spot Checks:**

Pick 3-5 random slide transitions and verify:

```bash
# Example: Check transition at 1:50 (Slide 3 → Slide 4)
# Open video, jump to 1:50
# Verify:
# - Correct slide is displayed
# - Narration matches slide content
# - Transition happens at natural speech pause
```

**Critical Transitions to Always Check:**
1. **First slide → Second slide** (0:30-0:45)
   - Usually intro → learning objectives

2. **Mid-video module transitions** (~3:00-5:00)
   - Topic changes, numbered lists

3. **Call to action / final slide** (last 2 minutes)
   - Summary → next steps

**Automated Verification:**

```bash
# Compare marker timestamps with actual narration
# For Slide 3 "Course Structure" at 1:17:
awk '/00:01:1[0-9]/{print; getline; print}' \
  transcripts/week0_commitment/0.5_VIDEO.srt

# Expected: Should mention "course structure" or "organization"
```

**Acceptance Criteria:**
- ✅ All slides transition within 10 seconds of ideal timing
- ✅ No slide displays for <5 seconds (too fast to read)
- ✅ No slide displays for >2 minutes without change
- ✅ Narration content matches visible slide content
- ✅ Total video duration matches source audio (±5 seconds)

---

## Complete Automation Script

Save as: `scripts/assemble_video_complete_workflow.sh`

```bash
#!/bin/bash
# Complete video assembly workflow with video-based SRT
# Usage: ./assemble_video_complete_workflow.sh 0.5_course_overview_expectations

set -e  # Exit on error

LESSON_ID="$1"
WEEK="${2:-week0_commitment}"

if [ -z "$LESSON_ID" ]; then
    echo "Usage: $0 <lesson_id> [week]"
    echo "Example: $0 0.5_course_overview_expectations week0_commitment"
    exit 1
fi

echo "========================================="
echo "Video Assembly - Complete Workflow"
echo "Lesson: $LESSON_ID"
echo "Week: $WEEK"
echo "========================================="

# Phase 1: Initial assembly
echo ""
echo "Phase 1/4: Creating TEMP video (equal time distribution)..."
uv run python scripts/assemble_video.py \
  --lesson "$LESSON_ID" \
  --week "$WEEK" \
  --type TEMP

TEMP_VIDEO="videos/$WEEK/${LESSON_ID}_TEMP.mp4"
if [ ! -f "$TEMP_VIDEO" ]; then
    echo "❌ TEMP video not created"
    exit 1
fi
echo "✓ TEMP video created: $TEMP_VIDEO"

# Phase 2: Generate SRT from video
echo ""
echo "Phase 2/4: Generating SRT from video..."
VIDEO_SRT="transcripts/$WEEK/${LESSON_ID}_VIDEO.srt"

~/opt/scripts/transcribe \
  "$TEMP_VIDEO" \
  --prompt "Please provide output in SRT subtitle format with timestamps" \
  -o "$VIDEO_SRT"

if [ ! -f "$VIDEO_SRT" ]; then
    echo "❌ Video SRT not created"
    exit 1
fi
echo "✓ Video-based SRT created: $VIDEO_SRT"

# Phase 3: Extract markers
echo ""
echo "Phase 3/4: Extracting slide markers from video-based SRT..."
SCRIPT_FILE="scripts/$WEEK/${LESSON_ID}.md"
MARKERS_JSON="transcripts/$WEEK/${LESSON_ID}_VIDEO_markers.json"

uv run python scripts/extract_markers_from_script.py \
  "$SCRIPT_FILE" \
  "$VIDEO_SRT"

if [ ! -f "$MARKERS_JSON" ]; then
    echo "❌ Markers JSON not created"
    exit 1
fi
echo "✓ Markers extracted: $MARKERS_JSON"

# Phase 4: Final assembly
echo ""
echo "Phase 4/4: Final assembly with corrected markers..."
FINAL_VIDEO="videos/$WEEK/${LESSON_ID}_FINAL.mp4"

# Use custom assembly script (adjust path as needed)
uv run python scripts/assemble_video_with_markers.py \
  --lesson "$LESSON_ID" \
  --markers "$MARKERS_JSON" \
  --output "$FINAL_VIDEO"

if [ ! -f "$FINAL_VIDEO" ]; then
    echo "❌ FINAL video not created"
    exit 1
fi

echo ""
echo "========================================="
echo "✅ SUCCESS - Video Assembly Complete"
echo "========================================="
echo "Output: $FINAL_VIDEO"
echo ""
echo "Next steps:"
echo "1. Verify video quality (spot check slide transitions)"
echo "2. git add $FINAL_VIDEO"
echo "3. git commit -m \"Add $LESSON_ID FINAL video\""
echo "4. git push"
echo ""
```

---

## Troubleshooting

### Issue: "transcribe script not found"

**Solution:**
```bash
ls -la ~/opt/scripts/transcribe
# If missing, check with overseer for installation
```

### Issue: SRT format not recognized

**Symptom:** Marker extraction fails with JSON parse error

**Solution:** Check SRT format is correct:
```bash
head -10 transcripts/week0_commitment/0.5_VIDEO.srt
# Should look like:
# 1
# 00:00:00,000 --> 00:00:05,000
# Text here
#
# 2
# 00:00:05,000 --> 00:00:10,000
# More text
```

### Issue: Marker extraction finds 0 markers

**Symptom:** `Found 0 slide markers`

**Solution:** Check script has markers:
```bash
grep -c "<!-- SLIDE:" scripts/week0_commitment/0.5_*.md
# Should return number > 0
```

### Issue: Video duration mismatch

**Symptom:** FINAL video is much shorter/longer than audio

**Solution:** Check you're using COMPLETE audio, not clipped:
```bash
ls -lh audio/week0_commitment/0.5_*.mp3
# COMPLETE version should be ~15MB for 16-min video
# SHORT/clipped version is ~2.4MB for 2-min video
```

### Issue: Slides transition too early/late

**Symptom:** Slide shows content not yet mentioned in narration

**Solution:** This is a marker extraction algorithm issue. The algorithm matches script text to transcript segments, but may pick an early/late occurrence of keywords.

**Workaround:** Manually adjust timestamps in the markers JSON:
```bash
# Edit markers JSON
nano transcripts/week0_commitment/0.5_VIDEO_markers.json

# Adjust specific timestamp (e.g., slide 4 from 110.48 to 119.32)
# Then re-run Phase 4 assembly
```

---

## Cost Analysis

**Per 16-minute video:**
- Phase 1 (TEMP assembly): $0 (local ffmpeg)
- Phase 2 (Video transcription): ~$0.10 (Gemini API)
- Phase 3 (Marker extraction): $0 (local processing)
- Phase 4 (FINAL assembly): $0 (local ffmpeg)

**Total: ~$0.10 per video**

**For 5 Week 0 videos: ~$0.50 total**

Compare to:
- Manual slide timing: 2-4 hours per video × $50/hr = $100-200/video
- Automated incorrect timing: Student confusion, support tickets, re-work

**ROI: Excellent** ✅

---

## Best Practices

### 1. Always Generate SRT from Video, Never from Audio
The source MP3 file has different timing than the video due to SSML processing.

### 2. Keep TEMP Videos for Debugging
Don't delete TEMP videos immediately. They're useful for:
- Re-generating SRT if needed
- Debugging timing issues
- Comparing with FINAL version

### 3. Commit Intermediate Files
Commit the video-based SRT and markers JSON to git:
```bash
git add transcripts/week0_commitment/0.5_VIDEO.srt
git add transcripts/week0_commitment/0.5_VIDEO_markers.json
```
This creates an audit trail and allows rollback if needed.

### 4. Spot Check Before Mass Production
Always verify one video completely before running the workflow on all 5 videos.

### 5. Document Timing Adjustments
If you manually adjust marker timestamps, document why:
```bash
# In commit message:
git commit -m "Adjust Slide 4 timing

Marker extraction placed slide at 1:50 (\"four hours\")
but should be at 1:59 (\"four main modules\").
Manually adjusted timestamp in markers JSON."
```

---

## Quick Reference

### File Naming Convention
```
0.X_lesson_name.md           # Script with markers
0.X_lesson_name.mp3          # Source audio
0.X_lesson_name.html         # Slides
0.X_lesson_name_TEMP.mp4     # Phase 1 output
0.X_lesson_name_VIDEO.srt    # Phase 2 output
0.X_lesson_name_VIDEO_markers.json  # Phase 3 output
0.X_lesson_name_FINAL.mp4    # Phase 4 output (DEPLOYABLE)
```

### One-Liner for Quick Assembly
```bash
cd /Users/tanwa/gt/psu_teaching/crew/video_producer/workshops/climate_policy && \
./scripts/assemble_video_complete_workflow.sh 0.5_course_overview_expectations
```

### Verify Video Is Ready for Deployment
```bash
# Check file exists and has reasonable size
ls -lh videos/week0_commitment/0.5_*_FINAL.mp4

# Check duration matches audio
ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 \
  videos/week0_commitment/0.5_*_FINAL.mp4

# Spot check 3 transitions manually
# Jump to: 0:37, 1:50, 7:05 in video player
```

---

## Success Metrics

A successful video assembly meets these criteria:

1. **Technical Quality**
   - ✅ 1920x1080 resolution
   - ✅ H.264 codec
   - ✅ AAC audio, 128 kbps
   - ✅ MP4 container

2. **Timing Quality**
   - ✅ Slide transitions within 10s of ideal timing
   - ✅ No slides <5s duration
   - ✅ No slides >120s duration
   - ✅ Total duration matches audio (±5s)

3. **Content Quality**
   - ✅ Narration matches visible slide content
   - ✅ Slide text readable at 1920x1080
   - ✅ Images/graphics display correctly
   - ✅ No visual glitches or artifacts

4. **Deployment Readiness**
   - ✅ Committed to git
   - ✅ Pushed to remote
   - ✅ QA verified
   - ✅ Ready for Moodle upload

---

## Version History

**v2.0 (2026-01-18)** - Video-Based SRT Workflow
- Generate SRT from TEMP video instead of source audio
- Eliminates 11-89s progressive timing drift
- Verified on 0.5 Course Overview video
- Status: ✅ PRODUCTION READY

**v1.0 (2026-01-16)** - Audio-Based SRT Workflow (DEPRECATED)
- Generated SRT from source MP3
- Had progressive timing drift issues
- Status: ❌ DO NOT USE

---

## Related Documentation

- `VIDEO_ASSEMBLY_WORKFLOW_CORRECTED.md` - Original investigation and solution
- `MARKER_BASED_SYNC_GUIDE.md` - Marker extraction details
- `scripts/assemble_video.py` - Core assembly tool
- `scripts/extract_markers_from_script.py` - Marker extraction tool

---

**Maintainer:** psu_teaching crew (video_producer, audio_engineer, qa_specialist)
**Last Updated:** 2026-01-18
**Status:** ✅ VERIFIED - Production ready
**Next Review:** After completing all Week 0 videos
