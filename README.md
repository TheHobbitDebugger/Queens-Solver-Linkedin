# Queens Solver and Chrome Bot

This project is a single-file helper for LinkedIn's Queens puzzle.

It has one main use:

- Generate a bot script from `index.html`.
- Run that script inside the live LinkedIn Queens page through Chrome DevTools.
- Let the script read the visible board, solve it, and click the missing queens.

The helper page cannot directly control another Chrome tab. Browsers block that
for security. Instead, `index.html` generates JavaScript that you paste into the
LinkedIn page itself, where the script is allowed to inspect and click that
page's DOM.

## Puzzle Logic

The solver represents the board as a square grid. Cells are stored in row-major
order:

- Cell `0` is row 1, column 1.
- Cell `1` is row 1, column 2.
- Cell `size` is row 2, column 1.

The bot scans each cell's visible background color and converts equal colors
into the same numeric region id. The solver then enforces the LinkedIn Queens
rules:

- Each row must contain exactly one queen.
- Each column must contain exactly one queen.
- Each colored region must contain exactly one queen.
- Queens may not touch diagonally. This is the LinkedIn rule; it is not the full
  chess diagonal attack rule.

The solver also supports existing queens already placed on the board. Existing
queens become fixed constraints. If those queens already conflict, solving stops
with an error.

## Solving Strategy

The pure solver is `solveQueens(...)` inside `index.html`.

It uses backtracking with small fast constraint sets:

- `assignment[row]` stores the selected column for that row, or `-1` if the row
  is still empty.
- `usedCols` tracks columns that already contain a queen.
- `usedRegions` tracks color regions that already contain a queen.
- Existing queens are stored in a row-to-column map before search starts.

At each step, the solver:

1. Finds every unfilled row.
2. Computes the legal columns for each row.
3. Chooses the row with the fewest legal columns.
4. Tries each legal column recursively.
5. Undoes a placement when it leads to a contradiction.
6. Stops when every row has one queen.

Choosing the row with the fewest legal columns is the minimum-remaining-values
heuristic. It keeps the search small because impossible or nearly forced rows
are handled early.

## Bot Architecture

Everything lives in `index.html`.

The file is split into these parts:

- HTML: title, rule summary, minimum-time input, copy buttons, generated code
  textarea, and status message.
- CSS: local helper-page styling only. The generated bot uses inline overlay
  styles so it does not depend on LinkedIn's CSS.
- `solveQueens(...)`: pure puzzle solver. It has no DOM dependency, so the same
  function can be embedded in the generated bot.
- `makeQueensBotSource()`: builds the self-contained script that runs inside
  LinkedIn.
- Copy helper: copy the raw script for the Chrome DevTools Console.

Inside the LinkedIn page, the generated bot runs this workflow:

1. Creates a small overlay in the top-right corner for progress messages.
2. Scans visible square-like DOM elements that might be board cells.
3. Clusters candidate elements by screen position into possible rows and columns.
4. Tests square grid sizes from 4 through 12.
5. Reads each candidate cell's background color to infer colored regions.
6. Prefers the largest valid grid whose color-region count matches the board
   size.
7. Reads existing queen or crown icons from text, ARIA labels, attributes, SVG
   hints, and CSS pseudo-content.
8. Calls `solveQueens(...)` with the scanned size, regions, and existing queens.
9. Clicks only the solution cells that do not already contain queens.

## Board Scanner Details

The scanner avoids relying on a single LinkedIn class name because the page DOM
can change.

It finds candidate cells using broad selectors such as:

- `button`
- `[role='button']`
- `[role='gridcell']`
- `[aria-label]`
- class names containing `cell`, `tile`, or `square`
- `[data-testid]`

If those selectors are not enough, it also scans all visible elements and keeps
only square-like rectangles.

For each possible grid, the scanner computes:

- cell centers
- grid bounds
- typical cell size
- size consistency between cells
- visible background color for every cell
- number of distinct color regions

This lets it reject unrelated page controls and choose the actual Queens board.

## Minimum Solve Time

The `Minimum seconds` input controls visible click pacing. The solver may find
the answer immediately, but the bot spaces out queen placements so the visible
solve takes at least the selected time.

For example, if you enter `10`, the bot schedules its clicks so the final queen
is placed near the 10-second mark.

## Launch the LinkedIn Bot

1. Open `index.html` in Chrome.
2. Set `Minimum seconds` if you want the bot to take a specific amount of time.
3. Click `Copy console bot`.
4. Open LinkedIn Queens in Chrome.
5. Make sure the full colored board is visible.
6. Open Chrome DevTools with `F12` or `Ctrl+Shift+I`.
7. Go to the `Console` tab.
8. Paste the copied script and press `Enter`.

## Troubleshooting

If the bot cannot find the board:

- Make sure the full colored grid is visible.
- Scroll until the board is not covered by popups, sticky headers, or overlays.
- Try a different browser zoom level.

If the bot says no solution was found:

- The scanner did find a grid, but the scanned colors or existing queens produced
  an impossible puzzle.
- Check the DevTools Console for `[Queens bot] Scanned region rows`.
- Send that region map plus the overlay message if the scan needs debugging.

If the bot finds the board but clicks the wrong cells:

- Make sure there are no accidental queens already placed on the board.
- Run the bot on a fresh/reset puzzle.
- Check whether the overlay reports the expected board size and region count.
