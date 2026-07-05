# Queens Solver and Chrome Bot

This folder contains a single-file helper for LinkedIn's Queens puzzle.

Open `index.html` in Chrome, click `Copy console bot`, then paste the copied
script into the DevTools Console on the LinkedIn Queens page.

The bot scans the visible colored board, groups cells by region color, solves the
Queens rules, and clicks the solution cells:

- One queen per row.
- One queen per column.
- One queen per colored region.
- No queens touching diagonally.

If the bot cannot find the board, make sure the full colored grid is visible and
not covered by popups or headers.
