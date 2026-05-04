# Search Feature Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a character search input to the grade selection screen that finds a Chinese character across all grades and navigates to its grade list with the matched tile highlighted.

**Architecture:** All changes are confined to `index.html` — the single-file SPA. A search input replaces the grade grid display while a query is active; results are rendered as `char-tile` elements with an added grade badge. `openGrade` gains an optional `highlightChar` parameter to pulse-highlight the matched tile after navigation.

**Tech Stack:** Vanilla HTML/CSS/JS, no build system. Open `index.html` directly in a browser to test.

---

## File Map

| File | Change |
|------|--------|
| `index.html` | Add HTML (search input + results div), CSS (input styles, results grid, pulse animation), JS (`handleSearch`, modified `openGrade`, input event listener) |

---

### Task 1: Add HTML — search input and results container

**Files:**
- Modify: `index.html` — inside `#screen-grades`, after `.app-subtitle`

- [ ] **Step 1: Add the input and results div**

Find this block in `index.html`:
```html
  <!-- Screen 1: Grade Selector -->
  <div id="screen-grades" class="screen active">
    <div class="app-title">🐼 Chinese Writer</div>
    <div class="app-subtitle">Pick your grade to start!</div>
    <div id="grade-grid"></div>
  </div>
```

Replace it with:
```html
  <!-- Screen 1: Grade Selector -->
  <div id="screen-grades" class="screen active">
    <div class="app-title">🐼 Chinese Writer</div>
    <div class="app-subtitle">Pick your grade to start!</div>
    <input id="search-input" type="text" placeholder="🔍 Search for a character..."
      oninput="handleSearch(this.value)">
    <div id="search-results"></div>
    <div id="grade-grid"></div>
  </div>
```

- [ ] **Step 2: Verify HTML in browser**

Open `index.html` in a browser. You should see a plain unstyled text input between the subtitle and the grade grid cards. The grade grid should still render normally.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add search input and results container to grade screen"
```

---

### Task 2: Add CSS — input styles, results grid, grade badge, highlight animation

**Files:**
- Modify: `index.html` — inside `<style>` block, after the `#cdnErrorBanner` rule

- [ ] **Step 1: Add the CSS**

Find the closing `</style>` tag and insert the following immediately before it:

```css
    #search-input {
      display: block;
      width: 100%;
      max-width: 420px;
      margin: 0 auto 16px;
      padding: 12px 16px;
      border: none;
      border-radius: 12px;
      font-family: 'Nunito', 'Arial Rounded MT Bold', sans-serif;
      font-size: 16px;
      font-weight: 600;
      background: white;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      outline: none;
    }
    #search-input:focus { outline: 3px solid #ff9a9e; }

    #search-results {
      display: none;
      grid-template-columns: repeat(4, 1fr);
      gap: 8px;
      max-width: 500px;
      margin: 0 auto;
    }

    .char-grade-badge {
      font-size: 9px;
      color: #ff9a9e;
      font-weight: 700;
      margin-top: 2px;
    }

    @keyframes highlight-pulse {
      0%   { box-shadow: 0 0 0 0   rgba(255, 202, 40, 0.9); }
      50%  { box-shadow: 0 0 0 8px rgba(255, 202, 40, 0.4); }
      100% { box-shadow: 0 0 0 0   rgba(255, 202, 40, 0);   }
    }
    .char-tile.highlighted { animation: highlight-pulse 0.8s ease-out; }
```

- [ ] **Step 2: Verify styles in browser**

Reload `index.html`. The search input should now look like a white rounded card (matching the app's style) with a pink focus ring when clicked. The grade grid should be unchanged.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add search input CSS, results grid, and highlight animation"
```

---

### Task 3: Add JS — `handleSearch` function

**Files:**
- Modify: `index.html` — inside `<script>` block, before `renderGradeSelector()`

- [ ] **Step 1: Add handleSearch**

Find the line `renderGradeSelector();` near the bottom of the `<script>` block and insert the following function above it:

```javascript
    function handleSearch(query) {
      const q = query.trim();
      const grid = document.getElementById('grade-grid');
      const results = document.getElementById('search-results');

      if (!q) {
        results.style.display = 'none';
        results.innerHTML = '';
        grid.style.display = '';
        return;
      }

      grid.style.display = 'none';

      const matches = [];
      GRADE_DATA.forEach(function(grade, gradeIndex) {
        grade.chars.forEach(function(c) {
          if (c.char === q) {
            matches.push({ grade: grade, gradeIndex: gradeIndex, c: c });
          }
        });
      });

      if (matches.length === 0) {
        results.style.display = 'block';
        results.innerHTML = '<p style="text-align:center;color:rgba(255,255,255,0.85);padding:20px;font-weight:700;">No results found</p>';
        return;
      }

      results.style.display = 'grid';
      results.innerHTML = '';
      matches.forEach(function(m) {
        var stars = getCharStars(m.c.char);
        var tile = document.createElement('div');
        tile.className = 'char-tile' + (stars === 0 ? ' unstarred' : '');
        tile.innerHTML =
          '<div class="char-glyph">' + m.c.char + '</div>' +
          '<div class="char-stars">' + (stars > 0 ? '⭐'.repeat(stars) : '·') + '</div>' +
          '<div class="char-grade-badge">G' + m.grade.grade + '</div>';
        tile.onclick = function() { openGrade(m.gradeIndex, m.c.char); };
        results.appendChild(tile);
      });
    }
```

- [ ] **Step 2: Verify in browser**

Reload `index.html`. Type a character that exists in the data (e.g. `花`) into the search box. The grade grid should disappear and result tiles should appear showing the character, its stars, and grade badge (e.g. "G1"). Clear the input — the grade grid should reappear. Type a character that doesn't exist (e.g. `龘`) — "No results found" should appear.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement handleSearch — filters all grades and renders result tiles"
```

---

### Task 4: Modify `openGrade` to accept optional `highlightChar` and pulse the matched tile

**Files:**
- Modify: `index.html` — the existing `openGrade` function in `<script>`

- [ ] **Step 1: Replace openGrade**

Find the existing `openGrade` function:
```javascript
    function openGrade(gradeIndex) {
      currentGrade = gradeIndex;
      renderCharacterList(gradeIndex);
      showScreen('chars');
    }
```

Replace it with:
```javascript
    function openGrade(gradeIndex, highlightChar) {
      currentGrade = gradeIndex;
      renderCharacterList(gradeIndex);
      showScreen('chars');

      if (highlightChar) {
        var chars = GRADE_DATA[gradeIndex].chars;
        var charIndex = -1;
        for (var i = 0; i < chars.length; i++) {
          if (chars[i].char === highlightChar) { charIndex = i; break; }
        }
        if (charIndex !== -1) {
          var tiles = document.getElementById('char-grid').querySelectorAll('.char-tile');
          var tile = tiles[charIndex];
          if (tile) {
            tile.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
            tile.classList.add('highlighted');
            setTimeout(function() { tile.classList.remove('highlighted'); }, 800);
          }
        }
      }
    }
```

- [ ] **Step 2: Verify in browser**

Reload `index.html`. Type `花` in the search box. Click the result tile. You should be taken to Grade 1's character list, and the 花 tile should briefly glow yellow before returning to normal. The existing grade card click behaviour (no highlight) should be unchanged.

- [ ] **Step 3: Verify existing openGrade callers still work**

Click any grade card directly from the grade screen (no search active). The grade character list should open normally with no errors in the browser console.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: highlight matched tile when navigating from search result"
```

---

### Task 5: End-to-end manual test pass

No code changes — verification only.

- [ ] **Step 1: Test search finds a character in one grade**

Type `山` (shān, mountain — Grade 1). One tile labelled "G1" should appear. Click it — lands on Grade 1 char list, 山 tile pulses yellow.

- [ ] **Step 2: Test search finds same character across multiple grades**

Type `爱` (ài, love). It appears in Grades 2, 3, and 5. Three tiles should appear: G2, G3, G5. Click each one and confirm each lands on the correct grade with the 爱 tile highlighted.

- [ ] **Step 3: Test no-match state**

Type `龘`. "No results found" message appears.

- [ ] **Step 4: Test clear restores grade grid**

Clear the search input (delete all text). Grade grid reappears with all 6 grade cards.

- [ ] **Step 5: Test grade card navigation still works**

Without using search, click Grade 1 directly. Character list opens normally, no console errors.

- [ ] **Step 6: Test CDN error banner still works**

No regression check needed — the search feature doesn't touch the CDN error path.

- [ ] **Step 7: Final commit if any tweaks were made**

```bash
git add index.html
git commit -m "fix: address issues found during end-to-end search testing"
```

Only run this step if you made changes during testing. Skip if all steps passed cleanly.
