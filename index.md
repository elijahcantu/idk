# Connections Plus Clone - Task Breakdown Document

## Project Overview
Build a word puzzle game where players group 16 words into 4 categories of 4 words each. Players select 4 words at a time and submit guesses to find the correct groupings.

## Core Game Data Structure
The puzzle data will be provided as:
```javascript
{
  puzzleId: number,
  date: string,
  words: [16 word strings],
  categories: [
    { name: string, difficulty: number, words: [4 words], color: string },
    { name: string, difficulty: number, words: [4 words], color: string },
    { name: string, difficulty: number, words: [4 words], color: string },
    { name: string, difficulty: number, words: [4 words], color: string }
  ]
}
```

---

## Component Architecture

### 1. App Component (Root)
**Purpose:** Main container managing overall application state and routing

**State:**
- `currentTheme`: 'light' | 'dark'
- `currentPuzzleId`: number
- `gameState`: object containing all game progress

**Props:** None (root component)

**Responsibilities:**
- Initialize theme from localStorage
- Manage navigation between different pages
- Hold global application state

---

### 2. Header Component
**Purpose:** Navigation bar with branding and menu options

**State:**
- None (receives all data via props)

**Props:**
- `theme`: 'light' | 'dark'
- `onThemeToggle`: function

**UI Elements:**
- Logo image (two versions: mini-logo.png for mobile, logo.png for desktop)
  - Mobile logo displays below 768px width
  - Desktop logo displays above 768px width
  - Both logos link to home page "/"
- Navigation links:
  - "NYT Archive" - links to "/nyt-archive"
  - "Community" - links to "/community" (not required to implement page)
  - "Create" button - styled as prominent button, links to "/create" (not required to implement page)
- Dark mode toggle button (moon/sun icon)
  - Shows moon icon in light mode
  - Shows sun icon in dark mode
  - Clicking toggles theme and saves to localStorage

---

### 3. GameContainer Component
**Purpose:** Main game area containing the puzzle

**State:**
- `selectedWords`: array of currently selected words (no maximum in multi-select mode)
- `selectionOrder`: array tracking order words were selected (for coloring)
- `isMultiSelectMode`: boolean (true when 5+ words selected)
- `foundCategories`: array of successfully found category objects
- `remainingWords`: array of words not yet correctly categorized
- `previousGuesses`: array of arrays storing all previous guess combinations
- `mistakes`: number (0-4)
- `gameStatus`: 'playing' | 'won' | 'lost'
- `isShuffling`: boolean for animation state
- `showResults`: boolean for showing final results

**Props:**
- `puzzleData`: object containing the puzzle information
- `puzzleId`: number

**Responsibilities:**
- Manage all game logic and state
- Track multi-select mode activation
- Store and check for duplicate guesses
- Coordinate child components
- Handle win/loss conditions

---

### 4. PuzzleHeader Component
**Purpose:** Display puzzle title and navigation

**State:**
- None

**Props:**
- `puzzleId`: number
- `date`: string (formatted as "Month DD, YYYY")

**UI Elements:**
- Left arrow button (links to previous puzzle /game/{puzzleId-1})
- Title text: "Connections #{puzzleId}"
- Date text below title
- Right arrow button (disabled if current puzzle, otherwise links to /game/{puzzleId+1})

---

### 5. WordGrid Component
**Purpose:** Display the 4x4 grid of word buttons

**State:**
- `animatingWords`: array of words currently animating
- `isMultiSelectMode`: boolean (true when 5+ words selected)

**Props:**
- `words`: array of word objects to display
- `selectedWords`: array of currently selected words
- `onWordClick`: function(word)
- `isShuffling`: boolean

**UI Elements:**
- 16 word buttons arranged in 4x4 grid
- Each button contains word text (may wrap to multiple lines if long)
- Selected words have colored backgrounds based on selection order
- During shuffle, words animate to new positions

**Interactions:**
- Normal Mode (0-4 words selected):
  - Clicking unselected word: adds to selection
  - Clicking selected word: removes from selection
  - After 4 words selected, hover remains enabled for multi-select activation
- Multi-Select Mode (5+ words selected):
  - Triggered by selecting a 5th word
  - Clicking any word toggles its selection
  - No maximum selection limit (can select all 16 words)
  - Clicking selected word removes it and may change colors of later selections

**Word Button States:**
- Default: light gray background
- Selected (Normal mode, 1-4 words): darker gray background with slight elevation
- Selected (Multi-select mode):
  - Words 1-4: Gray (original selection color)
  - Words 5-8: Blue background
  - Words 9-12: Purple background  
  - Words 13-16: Green background
- Hover: slightly darker shade (always enabled for unselected words)

**Multi-Select Color Logic:**
When in multi-select mode, track selection order and apply colors:
- Position 1-4 in selection array: Gray
- Position 5-8 in selection array: Blue
- Position 9-12 in selection array: Purple
- Position 13-16 in selection array: Green
- If a word is deselected, recolor remaining words based on new positions

---

### 6. CategoryDisplay Component
**Purpose:** Show successfully found categories

**State:**
- `isRevealing`: boolean for animation state

**Props:**
- `categories`: array of found category objects
- `difficulty`: number (1-4)

**UI Elements:**
- Horizontal bars that appear when category is found
- Each bar shows:
  - Category name (bold, centered)
  - Four words below (comma-separated)
- Color coding by difficulty:
  - Difficulty 1: Yellow (#F9DF6D)
  - Difficulty 2: Green (#A0C35A) 
  - Difficulty 3: Blue (#B0C4EF)
  - Difficulty 4: Purple (#BA81C5)

**Animation:**
- Slides in from top when revealed
- Brief shake animation if guess was one word away

---

### 7. MistakesIndicator Component
**Purpose:** Display remaining guesses

**State:**
- None

**Props:**
- `mistakes`: number (0-4)

**UI Elements:**
- 1 red heart icon and # of mistakes left to the right of it
- When the user runs out of mistakes (mistakes = 0), the heart and mistakes left are removed from view.

---

### 8. ControlButtons Component
**Purpose:** Action buttons for game control

**State:**
- None

**Props:**
- `selectedCount`: number
- `isMultiSelectMode`: boolean
- `onShuffle`: function
- `onDeselectAll`: function
- `onSubmit`: function
- `onShare`: function
- `isGameOver`: boolean

**UI Elements:**
- "Share" button (always visible)
  - Opens share modal/copies results to clipboard
- "Deselect All" button 
  - Disabled when no words selected
  - Clears all selections and exits multi-select mode
- "Shuffle" button
  - Triggers word position shuffle animation
  - Disabled during shuffle animation
- "Submit" button
  - Normal mode: Enabled only when exactly 4 words selected
  - Multi-select mode: Enabled only when exactly 4, 8, 12, or 16 words selected
  - Disabled if game is over
  - Darker prominent styling when enabled

**Special UI/Logic for Out of Mistakes:**
- When the user runs out of mistakes (mistakes = 0):
  - The heart and mistakes left are removed from the UI.
  - Below the word grid, display three buttons:
    - "Show Results" (shows the game over modal/results)
    - "Reveal Answer" (reveals all categories/answers)
    - "Continue Playing" (lets the user keep playing for fun)
- If the user clicks "Continue Playing":
  - The heart returns, but with an infinity symbol (∞) to the right of it instead of a number.
  - The options below the grid become:
    - "Reveal Answer"
    - "Deselect All"
    - "Shuffle"
    - "Submit"
  - The game continues in a free-play mode (no more mistake limit).

---

### 9. GameOverModal Component
**Purpose:** Display results when game ends

**State:**
- `isVisible`: boolean

**Props:**
- `gameStatus`: 'won' | 'lost'
- `foundCategories`: array
- `missedCategories`: array (only if lost)
- `onClose`: function
- `onShare`: function

**UI Elements:**
- Modal overlay (semi-transparent background)
- Modal content box:
  - Title: "Great!" (if won) or "Better luck next time!" (if lost)
  - List of found categories with their words
  - List of missed categories (if lost)
  - "Share Results" button
  - "Play Next Puzzle" button
  - Close button (X in corner)

---

### 10. Footer Component
**Purpose:** Links and copyright information

**State:**
- None

**Props:**
- None

**UI Elements:**
- "Buy Me A Coffee" link with coffee emoji
- "How To Play" link to "/how-to-play"
- "Privacy Policy" link to "/privacy"
- Copyright text: "© 2025 Connections Plus. All rights reserved. Connections Plus is not affiliated in any way with the New York Times."

---

## Game Logic and Interactions

### Word Selection Logic

**Normal Mode (0-4 selections):**
1. User clicks word button
2. If word is already selected: remove from selection
3. If word is not selected: add to selection
4. If now have 4 selections: ready to submit, but hover still enabled
5. Update selected words state

**Multi-Select Mode Activation:**
1. When 4 words are selected and user clicks a 5th word
2. Enter multi-select mode
3. Apply color coding based on selection order
4. Continue allowing selections up to all 16 words

**Multi-Select Mode Behavior:**
1. Any word can be clicked to toggle selection
2. Track selection order for color assignment:
   - Selections 1-4: Gray (original color)
   - Selections 5-8: Blue
   - Selections 9-12: Purple
   - Selections 13-16: Green
3. When word is deselected:
   - Remove from selection array
   - Recolor remaining words based on new positions
   - If selections drop below 5, exit multi-select mode
4. Submit button enabled only at 4, 8, 12, or 16 selections

### Submit Guess Logic

**Normal Mode (4 words selected):**
1. User clicks Submit
2. Check if this exact combination was already guessed (stored in `previousGuesses` array)
3. If duplicate guess:
   - Show notification: "Already guessed"
   - Shake all selected word blocks
   - Do NOT increment mistakes counter
   - Keep words selected
4. If new guess:
   - Add to `previousGuesses` array
   - Check if selected words match any category
5. If correct match:
   - Remove those words from grid
   - Add category to found categories (with reveal animation)
   - Clear selection
   - Check for win condition
6. If incorrect:
   - Shake all selected word blocks
   - Check if guess is "one away" (3 of 4 words correct)
   - If one away: show brief shake animation with message "One away!"
   - Increment mistakes counter by 1
   - Clear selection
   - Check for loss condition

**Multi-Select Mode (8, 12, or 16 words selected):**
1. User clicks Submit
2. Process selections in groups of 4 (based on selection order):
   - First 4 words (gray) checked as group 1
   - Words 5-8 (blue) checked as group 2
   - Words 9-12 (purple) checked as group 3
   - Words 13-16 (green) checked as group 4
3. Check each group for duplicates:
   - If any group is a duplicate guess: flag it but continue checking others
4. For each non-duplicate group of 4:
   - Add to `previousGuesses` array
   - Check if they match a category
   - If match: remove from grid and add to found categories
   - If no match: track as failed group
5. After processing all groups:
   - If all groups were duplicates:
     - Show "Already guessed" notification
     - Shake all word blocks
     - Do NOT increment mistakes
   - If at least one new guess was made:
     - If any groups matched: clear all selections, exit multi-select mode
     - If no groups matched:
       - Shake all selected word blocks
       - **Increment mistakes by 1 only** (regardless of how many groups failed)
       - Clear all selections
       - Exit multi-select mode
       - Check for loss condition
6. Show animations for all found categories simultaneously

### Shuffle Logic
1. User clicks Shuffle button
2. Set shuffling state to true
3. Randomly reorder remaining words (not found categories)
4. Animate words moving to new positions
5. Set shuffling state to false after animation

### Share Results Logic
1. Generate emoji grid based on game results:
   - Use colored squares for found categories (🟨🟩🟦🟪)
   - Order by when they were found
   - Include mistake count
2. Copy to clipboard
3. Show toast notification "Results copied!"

### Game Over Conditions
**Win:** All 4 categories found before 4 mistakes
**Loss:** 4 mistakes made before finding all categories

When the user runs out of mistakes (mistakes = 0):
1. The heart and mistakes left are removed from the UI.
2. Below the word grid, display buttons: "Show Results", "Reveal Answer", and "Continue Playing".
3. If the user clicks "Show Results": show the game over modal with results.
4. If the user clicks "Reveal Answer": reveal any unfound categories.
5. If the user clicks "Continue Playing":
  - The heart returns with an infinity symbol (∞) to the right of it.
  - The options below the grid become: "Reveal Answer", "Deselect All", "Shuffle", and "Submit".
  - The game continues in a free-play mode (no more mistake limit).
6. If the user finds all categories in free-play mode, the win modal is shown as usual.

---

## Duplicate Guess Handling

### Storage
- All submitted guesses stored in `previousGuesses` array
- Each guess stored as sorted array of 4 words (for easy comparison)
- Persists throughout game session

### Detection
Before processing any submission:
1. Sort the selected words alphabetically (for consistent comparison)
2. Check if this exact combination exists in `previousGuesses`
3. In multi-select mode, check each group of 4 independently

### User Feedback for Duplicates
**Visual Feedback:**
- All selected word blocks shake horizontally
- Same shake animation as incorrect guess
- Toast notification: "Already guessed"

**Game State:**
- Mistakes counter does NOT decrease
- Words remain selected (user can modify selection)
- No changes to game state or score

### Multi-Select Duplicate Handling
When submitting multiple groups:
- Each group checked individually for duplicates
- Only non-duplicate groups are processed
- If ALL groups are duplicates: no mistake penalty
- If ANY new group is submitted: normal processing (1 mistake if all new groups are wrong)

**Example Scenarios:**
1. Submit 8 words (2 groups): Group 1 is duplicate, Group 2 is new and wrong
   - Result: 1 mistake penalty for the new wrong group
2. Submit 8 words (2 groups): Both groups are duplicates
   - Result: No mistake penalty, "Already guessed" message
3. Submit 12 words (3 groups): 2 duplicates, 1 new and correct
   - Result: Correct group revealed, no mistake penalty

---

## Multi-Select Mode Details

### Activation
- Triggered when user has 4 words selected and clicks a 5th word
- Visual indicator: words change to colored backgrounds
- UI shows "Multi-Select Mode" indicator (optional)

### Color Coding System
Selection order determines colors:
- **Positions 1-4:** Gray (original selection color)
- **Positions 5-8:** Blue (#B0C4EF or similar)
- **Positions 9-12:** Purple (#BA81C5 or similar)
- **Positions 13-16:** Green (#A0C35A or similar)

### Deselection Behavior
When a word is deselected in multi-select mode:
1. Remove word from selection array
2. Shift all later selections forward
3. Reapply colors based on new positions
4. Example: If word at position 3 is deselected, word at position 5 moves to position 4 (changes from blue to gray)

### Submit Rules
- Submit enabled at exactly: 4, 8, 12, or 16 words
- Each group of 4 is evaluated independently
- **IMPORTANT: Single mistake penalty per submission** (even if multiple groups are wrong)
  - Example: Submitting 3 wrong groups = 1 mistake, not 3
- Duplicate guesses don't count as mistakes
- All correct groups are revealed simultaneously

### Exit Conditions
Multi-select mode exits when:
- User clicks "Deselect All"
- Selection count drops below 5 words
- User submits their guess (successful or not)
- Game ends

### Visual Feedback
- Color transitions should animate smoothly (0.2s)
- Submit button shows different state for multi-select
- Consider showing group boundaries visually
- May show text like "2 groups selected" when 8 words selected

---

## Responsive Design Requirements

### Mobile (< 768px)
- Header shows mini logo
- Word grid maintains 4x4 but smaller buttons
- Control buttons stack vertically
- Modal takes full screen width with padding

### Tablet (768px - 1024px)  
- Standard layout with adjusted spacing
- Word buttons medium size

### Desktop (> 1024px)
- Full logo in header
- Maximum container width of 1200px
- Larger word buttons with comfortable click targets

---

## Animation Requirements

1. **Word Selection:** Quick scale animation (0.1s)
2. **Category Reveal:** Slide down from top (0.3s)
3. **Wrong Guess Shake:** 
   - Horizontal shake animation (0.3s)
   - Applied to all selected words simultaneously
   - Same animation for duplicate guess and incorrect guess
4. **Duplicate Guess Feedback:**
   - Shake animation on word blocks
   - Toast notification "Already guessed" (appears for 2s)
   - No mistake counter animation
5. **Shuffle:** Words animate to new positions (0.4s)
6. **Modal Appearance:** Fade in overlay, scale in content (0.2s)
7. **Multi-Select Color Changes:** Smooth color transition (0.2s)
8. **Mistake Counter:** Heart empties with brief pulse animation (0.2s)

---

## State Persistence

Save to localStorage:
- Current theme preference
- Game progress for current puzzle (to allow refresh without losing progress)
- Statistics (games played, win %, streak)

---

## Error Handling

1. If puzzle data fails to load: Show error message with retry button
2. If submission fails: Show error toast, allow retry
3. Handle edge cases:
   - Rapid clicking during animations
   - Browser back/forward navigation
   - Network interruptions

---

## Accessibility Requirements

1. All interactive elements keyboard accessible
2. Proper ARIA labels for buttons and game state
3. Screen reader announcements for:
   - Word selection/deselection
   - Correct/incorrect guesses
   - Game over state
4. Focus management in modals
5. High contrast mode support

---

## Additional Pages (Non-Game)

### NYT Archive Page (/nyt-archive)
Display list of all available puzzles with:
- Puzzle number and date
- Completion status indicator
- Click to play any puzzle

### How To Play Page (/how-to-play)
Static page with:
- Game rules explanation
- Example puzzle walkthrough
- Tips and strategies

### Privacy Policy Page (/privacy)
Static page with standard privacy policy text

---

## Technical Implementation Notes

1. Use React Router for navigation
2. Use CSS Modules or styled-components for styling
3. Implement smooth animations with CSS transitions
4. Use React.memo for performance optimization on word buttons
5. Debounce rapid clicks to prevent animation conflicts
6. Lazy load puzzle data as needed
7. Implement proper loading states for all async operations

This completes the comprehensive task breakdown for the Connections Plus clone.