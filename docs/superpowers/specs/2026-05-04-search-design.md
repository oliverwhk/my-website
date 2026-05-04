# Search Feature Design

**Date:** 2026-05-04  
**Status:** Approved

## Summary

Add a character search input to the grade selection screen that lets students find a specific Chinese character across all grades and navigate directly to that character's grade list with the result highlighted.

## Requirements

- Search by exact character match (user types a Chinese character, e.g. 花)
- Search bar lives on `screen-grades`, between the subtitle and the grade grid
- Results replace the grade grid while the query is non-empty; clearing restores the grid
- Each result tile shows the character, its star progress, and a grade badge (e.g. "G1")
- The same character appearing in multiple grades shows as separate tiles
- Tapping a result navigates to that grade's char screen (`openGrade`) and highlights the matched tile with a pulse animation
- "No results" message when no characters match

## UI & Layout

```
🐼 Chinese Writer
Pick your grade to start!

[ 🔍 Search for a character... ]

[ Grade 1 ]  [ Grade 2 ]     ← hidden while query is non-empty
[ Grade 3 ]  [ Grade 4 ]
[ Grade 5 ]  [ Grade 6 ]
```

Results use the same 4-column `char-tile` grid as the char screen.

## Search Logic

- Match condition: `c.char === query.trim()`
- Searches all grades, returns all matches (duplicates across grades shown separately)
- No fuzzy matching, no pinyin, no meaning search — character only

## Navigation

Tapping a result tile:
1. Calls `openGrade(gradeIndex, highlightChar)` 
2. `renderCharacterList` runs as normal
3. After rendering, the matched tile gets `.highlighted` class → CSS pulse animation (yellow glow, ~0.8s)
4. Matched tile scrolled into view

## Implementation

### HTML (inside `screen-grades`)
- Add `<input id="search-input" type="text" placeholder="🔍 Search for a character...">`
- Add `<div id="search-results"></div>` below input, hidden by default

### CSS
- `#search-input`: white background, border-radius 12px, full width up to 420px, pink outline on focus, Nunito font
- `#search-results`: same 4-column grid as `#char-grid`, hidden by default
- `@keyframes highlight-pulse`: yellow box-shadow fade-out over 0.8s
- `.char-tile.highlighted`: applies `highlight-pulse` animation

### JS
- `handleSearch(query)`: filters `GRADE_DATA`, renders result tiles into `#search-results`, toggles `#grade-grid` / `#search-results` visibility
- `openGrade(gradeIndex, highlightChar)`: existing function gains optional `highlightChar` param; after `renderCharacterList`, finds and highlights the matching tile
- Wire `input` event on `#search-input` → `handleSearch`

### No changes to
- `openPractice`, HanziWriter lifecycle, localStorage, or any other screen
