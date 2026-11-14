# WorkedStats

**Homepage: [https://workedstats.com](https://workedstats.com)**

WorkedStats is a small, browser-only statistics workbench for students who need the working, not just the answer. Paste a dataset once, then move between the six tools below: the dataset is shared through `sessionStorage`, and every calculation and SVG plot runs in the tab.

- [Median Calculator](https://workedstats.com/) — sorts the data, shows the middle position or pair, and draws a box plot.
- [Mean Calculator](https://workedstats.com/tools/mean-calculator) — substitutes every value into Σx / n and compares the result with the median.
- [Standard Deviation Calculator](https://workedstats.com/tools/standard-deviation-calculator) — row-by-row deviation table, definition and computational formulas, and a sample/population switch.
- [Average Calculator](https://workedstats.com/tools/average-calculator) — an average-finder view with arithmetic, geometric, harmonic, midrange and mode.
- [Interquartile Range (IQR) Calculator](https://workedstats.com/tools/interquartile-range) — TI-84/AP exclusive, Tukey inclusive and Excel/R type-7 quartiles side by side, plus 1.5 × IQR fences.
- [Z-Score Calculator](https://workedstats.com/tools/z-score-calculator) — single-value and whole-dataset modes with a shaded standard-normal curve.

## Why it exists

Most statistics calculators return one number. WorkedStats keeps the intermediate work visible: the sorted list, the formula substitution, the deviation squares, the three quartile conventions that different courses use, and the same dataset across pages. There is no upload endpoint, no account, and no chart library.

The source for the live site is maintained separately; this repository is the public link and usage note.

A longer technical note on the three quartile conventions, worked deviation tables and inline SVG is on the blog: [Pure-front-end quartiles: TI-84, Tukey, Excel](https://xeon-space.com/posts/workedstats-browser-only-quartiles/).
