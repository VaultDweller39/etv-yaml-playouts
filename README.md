# ErsatzTV YAML Playout Reference Guide

## Overview

The ErsatzTV YAML playout system is a powerful, declarative scheduling engine that allows you to create sophisticated television channel programming using simple YAML configuration files. Instead of manually scheduling individual programs, you define content sources, create reusable instruction sequences, and let ErsatzTV handle the complex timing and scheduling automatically.

> **⚠️ BETA:** Please note the YAML Playouts are currently in **beta** and may change in future releases. Also I am not 100% sure the below is exactly correct.

This guide is a personally-maintained reference, intended to help early adopters experiment with YAML playouts. It is not official documentation and will eventually be retired once formal ErsatzTV documentation becomes available.

If you find inconsistencies or have improvements, please contribute via pull requests or share feedback in the ErsatzTV Discord.


## Revision History
- **2025-06-13**: Initial version created

## Table of Contents
- [Overview](#overview)
- [How It Works](#how-it-works)
- [Quick Reference](#quick-reference)
  - [Content Source Types](#content-source-types)
  - [Playout Instructions](#playout-instructions)
  - [Reset Instructions](#reset-instructions)




### How It Works
A YAML playout consists of four main components that work together:
1. **Content Sources** - Define where your media comes from
2. **Sequences** - Create reusable instruction blocks
3. **Reset Instructions** - Control startup and reset behavior  
4. **Playout Instructions** - Define the actual programming schedule

---

## Quick Reference

### Content Source Types
| Type | Purpose | Link |
|------|---------|------|
| `search` | Find content using search queries | [→ Search Sources](#search-content-sources) |
| `show` | Target specific shows by ID | [→ Show Sources](#show-content-sources) |
| `collection` | Use existing collections | [→ Collection Sources](#collection-content-sources) |
| `smart_collection` | Use smart collections | [→ Collection Sources](#collection-content-sources) |
| `multi_collection` | Use multi-collections | [→ Collection Sources](#collection-content-sources) |
| `playlist` | Use playlist content | [→ Collection Sources](#collection-content-sources) |
| `marathon` | Dynamic multi-show playlists | [→ Marathon Sources](#marathon-content-sources) |

### Playout Instructions
| Instruction | Purpose | Link |
|-------------|---------|------|
| `count` | Play X number of items | [→ Count Instructions](#count-instructions) |
| `duration` | Play content for X time | [→ Duration Instructions](#duration-instructions) |
| `all` | Play everything from source | [→ All Instructions](#all-instructions) |
| `pad_to_next` | Fill to next time interval | [→ Padding Instructions](#padding-instructions) |
| `pad_until` | Fill until specific time | [→ Padding Instructions](#padding-instructions) |
| `sequence` | Execute predefined sequence | [→ Sequence Instructions](#sequence-instructions) |
| `epg_group` | Control program guide entries | [→ EPG Grouping](#epg-grouping) |
| `repeat` | Loop back to start | [→ Control Flow](#control-flow) |

### Reset Instructions
| Instruction | Purpose | Link |
|-------------|---------|------|
| `wait_until` | Start at specific time | [→ Wait Instructions](#wait-instructions) |
| `skip_items` | Skip X items from source | [→ Skip Instructions](#skip-instructions) |
| `skip_to_item` | Jump to specific episode | [→ Skip Instructions](#skip-instructions) |

---

## Document Structure

Every YAML playout follows this basic structure:

```yaml
content:    # Define your media sources
  - search:
      key: "MY_CONTENT"
      query: "type:episode AND show_title:\"My Show\""
      order: "shuffle"

sequence:   # Optional: Create reusable instruction blocks
  - key: "MORNING_BLOCK"
    items:
      - count: 2
        content: "MY_CONTENT"

reset:      # Optional: Control startup behavior
  - wait_until: "6:00 AM"
    tomorrow: false

playout:    # Define your programming schedule
  - sequence: "MORNING_BLOCK"
  - repeat: true
```

---

## Content Sources

Content sources define where your media comes from. Each source needs a unique `key` that you'll reference in your playout instructions.

### Search Content Sources

Search sources use ErsatzTV's search engine to dynamically find content matching your criteria.

```yaml
content:
  - search:
    key: "SATURDAY_CARTOONS"
    query: "type:episode AND show_title:\"Looney Tunes\""
    order: "shuffle"
```

**Parameters:**
- `key` (required): Unique name to reference this content
- `query` (required): Search expression to find content
- `order` (required): `"chronological"` or `"shuffle"`

**Common Search Patterns:**
```yaml
# Find all episodes of a specific show
query: "type:episode AND show_title:\"The Simpsons\""

# Find movies from a specific decade
query: "type:movie AND release_date:[20000101 TO 20091231]"

# Find content with specific tags
query: "type:other_video AND tag:bumpers"

# Complex searches with multiple criteria
query: "type:episode AND (show_title:\"Friends\" OR show_title:\"Seinfeld\")"
```

> **💡 Testing Search Queries:** Before using search queries in your playout, test them in the ErsatzTV web application's search interface to make sure they return the content you expect. This helps avoid empty content sources or unexpected results in your schedule.

### Show Content Sources

Show sources target specific shows using their database identifiers, giving you precise control over content selection.

```yaml
content:
  - show:
    key: "FRIENDS_EPISODES"
    guids:
      - source: "tvdb"
        value: "79168"
      - source: "imdb"
        value: "tt0108778"
    order: "chronological"
```

**Parameters:**
- `key` (required): Unique name for this content source
- `guids` (required): List of show identifiers
- `order` (required): `"chronological"` or `"shuffle"`

**Supported Sources:**
- `tvdb` - The TV Database IDs
- `imdb` - Internet Movie Database IDs

### Collection Content Sources

Collection sources reference existing ErsatzTV collections, letting you reuse content you've already organized.

```yaml
content:
  # Regular Collection
  - collection: "Saturday Morning Cartoons"
    key: "CARTOONS"
    order: "shuffle"
  
  # Smart Collection (auto-updating)
  - smart_collection: "Recent Movies"
    key: "NEW_MOVIES"
    order: "chronological"
  
  # Multi Collection (combines multiple collections)
  - multi_collection: "All Comedy Shows"
    key: "COMEDY"
    order: "shuffle"
  
  # Playlist from Playlist Group
  - playlist: "My Favorites"
    playlist_group: "Personal Lists"
    key: "FAVORITES"
```

### Marathon Content Sources

Marathon sources create dynamic playlists that intelligently mix content from multiple shows or searches.

```yaml
content:
  - marathon:
    key: "COMEDY_MARATHON"
    # Option 1: Use specific show IDs
    guids:
      - source: "imdb"
        value: "tt0108778"  # Friends
      - source: "imdb"
        value: "tt0098904"  # Seinfeld
    
    # Option 2: Use search queries
    searches:
      - "type:episode AND show_title:\"Friends\""
      - "type:episode AND show_title:\"Seinfeld\""
      - "type:episode AND show_title:\"The Office\""
    
    group_by: "show"           # or "season"
    item_order: "shuffle"      # or "chronological"
    play_all_items: false     # Cycle through shows
    shuffle_groups: true      # Randomize show order
```

**How Marathons Work:**
- `group_by: "show"` - Groups content by show, then cycles between shows
- `group_by: "season"` - Groups by season, cycling between seasons
- `play_all_items: false` - Takes one item from each group before repeating
- `play_all_items: true` - Exhausts each group completely before moving to next
- `shuffle_groups: true` - Randomizes the order groups are selected

**Example Marathon Behavior:**
```yaml
# With play_all_items: false, shuffle_groups: true
# Might play: Friends S1E1 → Seinfeld S3E7 → The Office S2E4 → Friends S1E2 → ...

# With play_all_items: true, shuffle_groups: false  
# Might play: All Friends episodes → All Seinfeld episodes → All The Office episodes
```

---

## Sequences

Sequences let you create reusable blocks of instructions that can be executed multiple times throughout your playout.

```yaml
sequence:
  - key: "SHOW_WITH_BUMPERS"
    items:
      - count: 1
        content: "BUMPERS"
        filler_kind: "preroll"
      - count: 1
        content: "MAIN_SHOW"
      - count: 1
        content: "BUMPERS"
        filler_kind: "postroll"
```

**Benefits of Sequences:**
- **Consistency** - Ensure the same pattern is used everywhere
- **Maintainability** - Change the pattern in one place
- **Readability** - Give complex patterns meaningful names

**Common Sequence Patterns:**
```yaml
sequence:
  # Commercial break pattern
  - key: "COMMERCIAL_BREAK"
    items:
      - duration: "2 minutes"
        content: "COMMERCIALS"
        trim: true
  
  # Show block with intro/outro
  - key: "MORNING_SHOW_BLOCK"
    items:
      - count: 1
        content: "SHOW_INTROS"
      - count: 3
        content: "MORNING_SHOWS"  
      - count: 1
        content: "SHOW_OUTROS"
```

---

## Reset Instructions

Reset instructions control what happens when your playout starts up or resets. They run once at the beginning before your main playout instructions begin.

### Wait Instructions

Wait instructions pause the playout until a specific time, perfect for scheduling daily programming.

```yaml
reset:
  - wait_until: "6:00 AM"
    tomorrow: false
    rewind_on_reset: true
    offline_tail: false
```

**Parameters:**
- `wait_until` (required): Target time (supports 12/24 hour formats)
- `tomorrow` (required): If current time is past target, wait until tomorrow?
- `rewind_on_reset` (optional): Reset content sources to beginning on playout reset
- `offline_tail` (optional): Clock behavior for content that doesn't fill scheduled time

**Time Format Examples:**
```yaml
wait_until: "6:00 AM"    # 12-hour format
wait_until: "06:00"      # 24-hour format  
wait_until: "18:30"      # 6:30 PM
wait_until: "12:00pm"    # Noon
```

### Skip Instructions

Skip instructions let you start partway through your content sources, useful for resuming schedules or seasonal programming.

```yaml
reset:
  # Skip first 5 items
  - skip_items: 5
    content: "TV_SHOWS"
  
  # Jump to specific episode
  - skip_to_item:
      content: "SEASONAL_SHOW"
      season: 2
      episode: 10
```

---

## Playout Instructions

Playout instructions define your actual programming schedule. They execute in order and can loop indefinitely.

### Count Instructions

Count instructions play a specific number of items from a content source.

```yaml
playout:
  - count: 3
    content: "MORNING_CARTOONS"
    filler_kind: "none"
    custom_title: "Saturday Morning Block"
```

**Parameters:**
- `count` (required): Number of items to play
- `content` (required): Content source key to use
- `filler_kind` (optional): EPG classification - `"none"`, `"preroll"`, `"postroll"`
- `custom_title` (optional): Override EPG title

**Use Cases:**
```yaml
# Play exactly 2 episodes
- count: 2
  content: "SITCOMS"

# Play 1 bumper as preroll content
- count: 1
  content: "BUMPERS"
  filler_kind: "preroll"
```

### Duration Instructions

Duration instructions play content for a specific amount of time, automatically selecting items that fit.

```yaml
playout:
  - duration: "30 minutes"
    content: "SHORT_SHOWS"
    trim: true
    discard_attempts: 3
    fallback: "FILLER_CONTENT"
```

**Parameters:**
- `duration` (required): Time to fill (e.g., "30 minutes", "1 hour")  
- `content` (required): Content source to use
- `trim` (optional): Cut content to fit exactly?
- `discard_attempts` (optional): How many items to skip if they don't fit
- `fallback` (optional): Content to use if nothing else fits

**How Duration Works:**
1. ErsatzTV picks the next item from your content source
2. If it fits in the time slot, it plays
3. If it doesn't fit and `trim: false`, it tries the next item (up to `discard_attempts`)
4. If `trim: true`, it cuts the content to fit exactly
5. If nothing fits, it uses `fallback` content (looped and trimmed as needed)

### All Instructions

All instructions play every item from a content source before continuing.

```yaml
playout:
  - all:
    content: "SPECIAL_EVENT"
```

**Use Case:** Perfect for special events, movie marathons, or when you want to ensure all content in a source gets played.

### Padding Instructions

Padding instructions fill time until a specific condition is met, essential for maintaining schedule timing.

#### Pad to Next Interval

```yaml
playout:
  - pad_to_next: 30  # minutes
    content: "FILLER"
    trim: true
    discard_attempts: 2
    fallback: "BACKUP_FILLER"
    filler_kind: "postroll"
```

**Use Case:** Fill time until the next half-hour or hour mark.

#### Pad Until Specific Time

```yaml
playout:
  - pad_until: "9:00 AM"
    tomorrow: false
    content: "MORNING_FILLER"
    trim: true
    fallback: "BACKUP_CONTENT"
```

**Use Case:** Fill time until a specific scheduled event.

### Sequence Instructions

Sequence instructions execute predefined sequence blocks.

```yaml
playout:
  # Execute sequence in order
  - sequence: "MORNING_BLOCK"
  
  # Execute sequence with shuffled order
  - shuffle_sequence: "VARIETY_BLOCK"
```

### EPG Grouping

EPG (Electronic Program Guide) grouping controls how content appears in program guides and how shows are logically grouped together.

**Why Use EPG Groups?**

Without EPG groups, every single playout instruction creates its own program guide entry. This means viewers would see separate entries for:
- Each individual bumper
- Each commercial break  
- Each piece of filler content
- Every single item you play

This creates a cluttered, confusing program guide where a single 30-minute show might appear as 5+ separate entries.

**The Problem EPG Groups Solve:**

Imagine you want to create a "Saturday Morning Cartoons" block that includes:
1. Opening bumper
2. 3 cartoon episodes 
3. Commercial breaks between episodes
4. Closing bumper

Without EPG groups, your program guide would show:
- 9:00 AM - "Bumper"
- 9:01 AM - "Tom and Jerry S1E1" 
- 9:08 AM - "Commercial"
- 9:10 AM - "Bugs Bunny S2E5"
- 9:17 AM - "Commercial" 
- 9:19 AM - "Scooby Doo S1E3"
- 9:26 AM - "Bumper"

With EPG groups, viewers see:
- 9:00 AM - "Saturday Morning Cartoons" (30 minutes)

**How to Use EPG Groups:**

```yaml
playout:
  - epg_group: true        # Start new program entry
    advance: true          # Create new EPG entry (default)
  
  - count: 1
    content: "BUMPERS" 
    filler_kind: "preroll"  # Won't appear in EPG
  
  - count: 1
    content: "MAIN_SHOW"    # Generates EPG metadata
  
  - count: 1
    content: "BUMPERS"
    filler_kind: "postroll" # Won't appear in EPG
  
  - epg_group: false       # End program entry
```

**How EPG Grouping Works:**

**Without EPG Groups (default):**
- Each playout instruction creates its own program guide entry
- Viewers see every individual piece of content as a separate program

**With EPG Groups:**
- `epg_group: true` starts a new program entry
- All content until `epg_group: false` becomes part of that single program
- Only content with `filler_kind: "none"` generates program metadata
- `preroll` and `postroll` content is hidden from the guide

**Advanced EPG Control:**
```yaml
# Create new program entry
- epg_group: true
  advance: true

# Lock current program entry (don't create new one)  
- epg_group: true
  advance: false
```

**Real-World Example:**
```yaml
# This creates ONE program guide entry called "Comedy Hour"
# that runs for however long all this content takes
- epg_group: true
  custom_title: "Comedy Hour"

- count: 1
  content: "SHOW_OPEN_BUMPER"
  filler_kind: "preroll"      # Hidden from guide

- count: 2  
  content: "FRIENDS"          # Provides show metadata for guide

- duration: "3 minutes"
  content: "COMMERCIALS" 
  filler_kind: "postroll"     # Hidden from guide

- count: 2
  content: "SEINFELD"         # More show content

- count: 1
  content: "SHOW_CLOSE_BUMPER"
  filler_kind: "postroll"     # Hidden from guide

- epg_group: false            # End "Comedy Hour" program
```

### Control Flow

Control flow instructions manage the execution flow of your playout.

```yaml
playout:
  - count: 1
    content: "SHOWS"
  - repeat: true  # Go back to beginning of playout
```

---

## Advanced Features

### Common Parameters

These parameters can be used with multiple instruction types:

**Filler Kind Classification:**
- `filler_kind: "none"` - Primary content (appears in EPG)
- `filler_kind: "preroll"` - Content before main program (hidden from EPG)  
- `filler_kind: "postroll"` - Content after main program (hidden from EPG)

**Content Fitting Options:**
- `trim: true` - Cut content to fit time slots exactly
- `discard_attempts: 3` - Try this many items before giving up
- `fallback: "BACKUP_KEY"` - Use this content when nothing else fits

**Time and Reset Options:**
- `tomorrow: true` - Wait until tomorrow if time has passed
- `rewind_on_reset: true` - Reset content sources on playout reset
- `offline_tail: false` - Don't advance clock beyond scheduled content

### Search Query Syntax

ErsatzTV supports powerful search expressions:

```yaml
# Basic filters
"type:episode"                          # Only TV episodes
"type:movie"                           # Only movies  
"type:other_video"                     # Other video content

# Metadata searches
"show_title:\"The Simpsons\""          # Exact show title
"tag:commercials"                      # Content with specific tag
"genre:comedy"                         # Content in comedy genre

# Date ranges
"release_date:[20000101 TO 20091231]"  # 2000s content
"air_date:[20200101 TO *]"             # Content after 2020

# Boolean logic
"type:episode AND show_title:\"Friends\""
"(show_title:\"Friends\" OR show_title:\"Seinfeld\") AND NOT tag:pilot"
```

> **💡 Testing Search Queries:** Before using search queries in your playout, test them in the ErsatzTV web application's search interface to make sure they return the content you expect. This helps avoid empty content sources or unexpected results in your schedule.

---

## Complete Examples

### Example 1: Simple Morning Cartoon Block

A basic Saturday morning cartoon schedule with bumpers between shows.

```yaml
content:
  - search:
    key: "CARTOONS"
    query: "type:episode AND (show_title:\"Tom and Jerry\" OR show_title:\"Looney Tunes\")"
    order: "shuffle"
  
  - search:
    key: "BUMPERS"
    query: "type:other_video AND tag:bumpers"
    order: "shuffle"
  
  - search:
    key: "FILLER"
    query: "type:other_video AND tag:filler"
    order: "shuffle"

sequence:
  - key: "CARTOON_BLOCK"
    items:
      - count: 1
        content: "BUMPERS"
        filler_kind: "preroll"
      - count: 2
        content: "CARTOONS"
      - count: 1
        content: "BUMPERS"
        filler_kind: "postroll"

reset:
  - wait_until: "7:00 AM"
    tomorrow: false

playout:
  - epg_group: true
  - sequence: "CARTOON_BLOCK"
  - pad_to_next: 60  # Fill to next hour
    content: "FILLER"
    trim: true
    filler_kind: "postroll"
  - epg_group: false
  - repeat: true
```

### Example 2: Complex Multi-Show Schedule

A sophisticated schedule mixing different types of content throughout the day.

```yaml
content:
  # Morning shows
  - marathon:
    key: "MORNING_SHOWS"
    searches:
      - "type:episode AND show_title:\"The Smurfs\""
      - "type:episode AND show_title:\"Scooby-Doo\""
      - "type:episode AND show_title:\"Flintstones\""
    group_by: "show"
    item_order: "shuffle"
    play_all_items: false
    shuffle_groups: true
  
  # Afternoon sitcoms
  - marathon:
    key: "SITCOMS"
    guids:
      - source: "imdb"
        value: "tt0108778"  # Friends
      - source: "imdb"
        value: "tt0098904"  # Seinfeld
    group_by: "show"
    item_order: "chronological"
    play_all_items: false
  
  # Evening movies
  - search:
    key: "MOVIES"
    query: "type:movie AND release_date:[19800101 TO 19991231]"
    order: "shuffle"
  
  # Filler content
  - search:
    key: "BUMPERS"
    query: "type:other_video AND tag:bumpers"
    order: "shuffle"
  
  - search:
    key: "COMMERCIALS"
    query: "type:other_video AND tag:commercials"
    order: "shuffle"
  
  - search:
    key: "LATE_NIGHT_FILLER"
    query: "type:other_video AND tag:latenight"
    order: "shuffle"

sequence:
  - key: "MORNING_BLOCK"
    items:
      - count: 1
        content: "BUMPERS"
        filler_kind: "preroll"
      - count: 4
        content: "MORNING_SHOWS"
      - duration: "3 minutes"
        content: "COMMERCIALS"
        filler_kind: "postroll"
        trim: true
  
  - key: "SITCOM_BLOCK"
    items:
      - count: 1
        content: "BUMPERS" 
        filler_kind: "preroll"
      - count: 2
        content: "SITCOMS"
      - duration: "2 minutes"
        content: "COMMERCIALS"
        filler_kind: "postroll"
        trim: true

reset:
  - wait_until: "6:00 AM"
    tomorrow: false
    rewind_on_reset: true

playout:
  # 6:00 AM - 12:00 PM: Morning cartoons
  - epg_group: true
  - sequence: "MORNING_BLOCK"
  - pad_until: "12:00 PM"
    tomorrow: false
    content: "MORNING_SHOWS"
    filler_kind: "none"
  - epg_group: false
  
  # 12:00 PM - 6:00 PM: Afternoon sitcoms  
  - epg_group: true
  - sequence: "SITCOM_BLOCK"
  - pad_until: "6:00 PM"
    tomorrow: false
    content: "SITCOMS"
    filler_kind: "none"
  - epg_group: false
  
  # 6:00 PM - 10:00 PM: Evening movies
  - epg_group: true
  - count: 1
    content: "MOVIES"
  - pad_until: "10:00 PM"
    tomorrow: false
    content: "LATE_NIGHT_FILLER"
    trim: true
    filler_kind: "postroll"
  - epg_group: false
  
  # 10:00 PM - 6:00 AM: Late night filler
  - duration: "8 hours"
    content: "LATE_NIGHT_FILLER"
    trim: false
  
  - repeat: true
```

### Example 3: Special Event Marathon

A focused marathon for a special event or holiday programming.

```yaml
content:
  - show:
    key: "HALLOWEEN_SPECIAL"
    guids:
      - source: "tvdb"
        value: "12345"  # Halloween-themed show
    order: "chronological"
  
  - search:
    key: "HORROR_MOVIES"
    query: "type:movie AND genre:horror AND rating:[1 TO 3]"  # Family-friendly horror
    order: "shuffle"
  
  - search:
    key: "SPOOKY_EPISODES"
    query: "type:episode AND (tag:halloween OR tag:spooky)"
    order: "shuffle"
  
  - marathon:
    key: "MIXED_SPOOKY"
    searches:
      - "type:episode AND show_title:\"Scooby-Doo\""
      - "type:episode AND show_title:\"The Addams Family\""  
      - "type:episode AND show_title:\"The Munsters\""
    group_by: "show"
    item_order: "shuffle"
    play_all_items: false
    shuffle_groups: true
  
  - search:
    key: "HALLOWEEN_BUMPERS"
    query: "type:other_video AND tag:halloween_bumpers"
    order: "shuffle"

sequence:
  - key: "SPOOKY_BLOCK"
    items:
      - count: 1
        content: "HALLOWEEN_BUMPERS"
        filler_kind: "preroll"
      - count: 3
        content: "MIXED_SPOOKY"
      - count: 1
        content: "HALLOWEEN_BUMPERS"
        filler_kind: "postroll"

reset:
  - wait_until: "12:00 AM"  # Start at midnight
    tomorrow: false

playout:
  # Midnight - 6 AM: Mixed spooky content
  - epg_group: true
    custom_title: "All Night Spook-a-thon"
  - sequence: "SPOOKY_BLOCK"
  - pad_until: "6:00 AM"
    tomorrow: true
    content: "SPOOKY_EPISODES"
    filler_kind: "none"
  - epg_group: false
  
  # 6 AM - 6 PM: Halloween specials and family content
  - epg_group: true
    custom_title: "Halloween Family Fun"
  - all:
    content: "HALLOWEEN_SPECIAL"
  - count: 5
    content: "SPOOKY_EPISODES"
  - pad_until: "6:00 PM" 
    tomorrow: false
    content: "MIXED_SPOOKY"
  - epg_group: false
  
  # 6 PM - Midnight: Horror movie night
  - epg_group: true
    custom_title: "Fright Night Movies"
  - count: 2
    content: "HORROR_MOVIES"
  - pad_until: "11:59 PM"
    tomorrow: false
    content: "SPOOKY_EPISODES"
    trim: true
  - epg_group: false
  
  - repeat: true
```

---

## HINTS AND TIPS

### Content Organization
- **Use descriptive keys** - `"SATURDAY_CARTOONS"` instead of `"CONTENT1"`
- **Group related content** - Keep similar content sources together
- **Plan for fallbacks** - Always have backup content for padding operations

### EPG Management
- **Group logically related content** - Use EPG groups to create meaningful program entries
- **Use filler classifications** - Mark bumpers and commercials as `preroll`/`postroll`
- **Provide custom titles** - Give EPG entries descriptive names

### Schedule Design
- **Start with simple patterns** - Build complexity gradually
- **Test timing carefully** - Use padding instructions to maintain schedule
- **Plan for edge cases** - What happens when content runs out?

### Performance Tips
- **Use specific searches** - Narrow queries perform better than broad ones
- **Limit discard attempts** - Don't let duration instructions search forever
- **Monitor content sources** - Ensure your sources have enough content

