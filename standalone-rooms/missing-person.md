# TryHackMe — Missing Person

## Overview

**Platform:** TryHackMe  
**Category:** OSINT / Digital Forensics  
**Core techniques:** image analysis, EXIF metadata, reverse-image search, map validation, event-timeline correlation, social-media pivoting, cross-platform identity resolution

Missing Person is an OSINT investigation built around correlating weak signals from multiple public sources. The room is valuable because no single clue solves the case. The investigator must move from photographs to places, places to dates, dates to events, and events to people.

Direct challenge answers and personal contact values are intentionally excluded from this public writeup.

## Investigation Model

```text
Provided images
    ↓
Visual landmarks and branding
    ↓
Reverse-image / map validation
    ↓
Event timeline
    ↓
EXIF metadata
    ↓
Venue identification
    ↓
Social-media posts and tags
    ↓
Cross-platform identity pivot
    ↓
Public business/contact evidence
```

## 1. Preserve the Evidence

Before analysis, keep the original files unchanged.

```bash
mkdir evidence
cp *.jpg evidence/
sha256sum evidence/*
```

Record hashes so later edits, conversions, or metadata extraction do not silently replace the originals.

For a real investigation, maintain:

- source filename
- acquisition time
- SHA-256
- origin/source
- analyst notes

## 2. Visual Image Analysis

Start with what can be seen without tools:

- logos
- sponsor names
- road markings
- architecture
- menus
- signs
- language
- clothing
- weather
- vehicle plates
- geographic features

Crop distinctive areas before reverse-image searching. A focused crop usually produces better results than uploading the entire image.

Useful search patterns include:

```text
"<visible brand>" circuit Indonesia
"<restaurant branding>" Lombok
"<event name>" after party Kuta
```

### Validation rule

Never accept the first visually similar result. Verify with at least two independent attributes such as:

- matching building geometry
- circuit layout
- event schedule
- location
- sponsor branding
- business photos

## 3. EXIF Analysis

Use ExifTool against the original artifact:

```bash
exiftool image.jpg
```

Fields worth reviewing:

```text
Date/Time Original
Create Date
Modify Date
GPS Latitude
GPS Longitude
Make
Model
Software
Orientation
```

A targeted command:

```bash
exiftool -DateTimeOriginal -CreateDate -GPSLatitude -GPSLongitude image.jpg
```

### Important limitation

EXIF is evidence, not truth.

Metadata can be:

- stripped by social networks
- rewritten by editors
- modified intentionally
- copied from another file

Correlate EXIF timestamps with the external event timeline instead of using metadata alone.

## 4. Timeline Construction

Once the geographic area is identified, build a timeline around public events.

Example analyst worksheet:

| Time | Evidence | Confidence | Source |
|---|---|---:|---|
| Day 1 | Image associated with motorsport venue | Medium | Photo |
| Day 1 | Official event calendar matches location | High | Event source |
| Evening | Restaurant image timestamp | Medium/High | EXIF |
| Night | Venue promotion for after-party | Medium | Social media |
| Next day | Guide/tour activity | Medium | Cross-platform profile |

This makes contradictions visible and prevents the investigation from becoming a collection of disconnected browser tabs.

## 5. Venue and Event Pivoting

Once an event and date range are established, search:

- venue Instagram/Facebook pages
- event flyers
- tagged photos
- reels
- maps/business listings
- public reviews
- cached/archived pages

Useful queries:

```text
"<event>" "<city>" after party
site:instagram.com "<venue>" "<event>"
site:facebook.com "<venue>" "<month year>"
```

The goal is to identify the relevant venue and performers without assuming that the most prominent artist or headliner is the person referenced by the scenario.

## 6. Social-Media Identity Resolution

When a performer or local contact is found, build an identity graph.

```text
Stage name
   ↓
Social username
   ↓
Profile image / bio
   ↓
Other platform
   ↓
Business or tour page
   ↓
Location / contact evidence
```

Correlate:

- same/similar username
- same profile photo
- overlapping geographic area
- repeated business name
- matching phone format
- matching venue/event relationships

One matching username is not enough to establish identity.

## 7. OSINT Confidence Scoring

A simple confidence model improves discipline:

**High confidence**
- official business page + map record + matching imagery

**Medium confidence**
- social post + matching location/date + consistent identity

**Low confidence**
- single repost, AI-generated search summary, anonymous forum comment

Record why a conclusion is trusted, not only what the conclusion is.

## 8. Privacy and Responsible Handling

This type of room deliberately uses public information, but the real-world equivalent can involve sensitive personal data.

For public documentation:

- redact phone numbers
- redact unrelated personal email addresses
- avoid republishing private photos
- do not encourage contacting real individuals
- separate lab evidence from real-world targeting

## Useful Tooling

```bash
exiftool <file>
sha256sum <file>
file <file>
strings <file> | less
```

Additional tools:

- Google Lens or equivalent reverse-image search
- Google Maps / OpenStreetMap
- Wayback Machine
- search-engine operators
- username correlation tools used within legal/ethical boundaries

## Lessons Learned

The room reinforces several OSINT principles:

1. Visual clues are often more valuable than filenames.
2. Metadata is strongest when correlated with independent evidence.
3. Public event schedules provide excellent timeline anchors.
4. Social tags create relationship graphs.
5. Cross-platform identity resolution requires multiple matching signals.
6. Good OSINT work records confidence and provenance, not just answers.

## References

- TryHackMe Missing Person room: https://tryhackme.com/room/missingperson
