# Architecture: descriptive statistics without a calculator API

WorkedStats is deliberately small on the server and busy in the browser. The deployed Worker serves the page and its assets; it does not receive a dataset, run a statistics library, or store a result. That boundary is the architecture.

The public tools are:

- [Median Calculator](https://workedstats.com/) for the middle value and a box plot.
- [Mean Calculator](https://workedstats.com/tools/mean-calculator) for Σx / n with the substitution.
- [Standard Deviation Calculator](https://workedstats.com/tools/standard-deviation-calculator) for the deviation table and both SD formulas.
- [Average Calculator](https://workedstats.com/tools/average-calculator) for the average family.
- [Interquartile Range (IQR) Calculator](https://workedstats.com/tools/interquartile-range) for the three quartile conventions and fences.
- [Z-Score Calculator](https://workedstats.com/tools/z-score-calculator) for a value or a whole dataset with normal probabilities.

## Request path

A tool page is a React 19 route rendered by TanStack Start on Cloudflare Workers. The route loader selects a tool spec: title, H1, copy, limits, FAQ and the calculator kind. The page renders a textarea and the calculator component. There is no API route behind the result.

```
Pasted text
  -> parse numbers (commas / spaces / tabs / newlines / Excel column)
  -> sessionStorage (shared across the six pages)
  -> pure statistics functions
  -> worked steps + summary + inline SVG
```

The statistics path makes no network requests. Loading the page fetches its own assets; the calculator itself does not. The result is derived synchronously from the parsed numbers.

## The three quartile conventions

Q1 and Q3 are not one algorithm. The page deliberately computes three:

- **Exclusive (TI-83/84, AP Statistics)** — for odd `n`, remove the median before splitting the sorted list into halves; Q1 and Q3 are the medians of those halves.
- **Inclusive (Tukey's hinges)** — for odd `n`, keep the median in both halves; Q1 and Q3 are the medians of the overlapping halves.
- **Linear interpolation (Excel `QUARTILE.INC`, R type 7, NumPy default)** — Q1 and Q3 are values at position `(n − 1)p`, interpolated between adjacent observations. They need not be observed values.

For `12, 15, 18, 20, 22, 25, 28, 31, 35`:

| Convention | Q1 | Q3 | IQR |
| --- | ---: | ---: | ---: |
| Exclusive | 16.5 | 29.5 | 13 |
| Inclusive | 18 | 28 | 10 |
| Linear type 7 | 18 | 28 | 10 |

The IQR page shows all three cards at once. The active card controls the 1.5 × IQR fences and the box plot, so changing the convention can move a point inside or outside the fence. That is the lesson, not an inconsistency.

## Standard deviation, two ways

The SD page sorts the data and computes the mean. It then builds a table with one row per observation:

```
xᵢ | xᵢ − x̄ | (xᵢ − x̄)²
```

The footer is `Σ(xᵢ − x̄)²`. The same sum appears in both formulas:

```
s = √(Σ(xᵢ − x̄)² / (n − 1))            definition
s = √((Σxᵢ² − n·x̄²) / (n − 1))          computational
```

The sample/population toggle changes the denominator from `n − 1` to `N`. The raw sum of squares does not change; only the variance and SD do. The APA button copies `M = …, SD = …` using the sample SD, because that is the usual reporting convention.

## Shared dataset & interactive workbench

The six pages read and write the same `sessionStorage` key. On a fresh tab with no stored dataset, the workbench pre-loads the ten-value outlier preset (`12, 15, 18, 20, 22, 25, 28, 31, 35, 98`) so visitors immediately see a populated box plot and worked steps; clearing the textarea persists an empty string across navigation. The parser extracts numeric tokens and ignores non-numeric text, so a pasted Excel column works without a header cleanup step.

On desktop viewports the input rail stays sticky beside the result canvas, letting users toggle individual sorted token chips to exclude or restore outliers and switch decimal precision (`Auto`, `2`, `4`) without scrolling away from the charts or formulas.

This is a UX choice with a statistical consequence. Comparing the mean with the median, then checking the IQR, then testing how removing `98` changes every metric is one investigation. Keeping the dataset in the tab removes the retyping that would otherwise break it into four unrelated visits.

## SVG instead of a chart library

The box plot is a set of `<line>`, `<rect>` and `<circle>` elements. The x scale runs from the dataset minimum to its maximum. The box spans Q1 to Q3; the median is a vertical line; whiskers stop at the most extreme non-outlier. Values outside the fences get a separate red circle and a `<title>`.

The normal curve is generated from the standard normal PDF at 160 points between −4 and 4. The selected tail is filled with one or two area paths. The CDF uses the Abramowitz & Stegun 7.1.26 approximation, with an error around `1.5 × 10⁻⁷`:

```ts
export function normalCdf(z: number): number {
  return 0.5 * (1 + erf(z / Math.SQRT2));
}
```

The charts are in the DOM, so they are screenshot-friendly and accessible through `<title>` and `<desc>`. There is no chart package in the bundle.

## Testing

The pure functions live in `src/lib/stats.ts` and are covered by Vitest: parsing, medians, quartile conventions, fences, sample/population SD, modes and the normal CDF. A Playwright script drives all six pages in Chromium: the shared dataset across routes, the SD table and toggle, APA copy, the three IQR cards, the outlier dot, the bell curve tails, and the browser console.

The test that matters most for this product is the cross-page one: paste on the median page, navigate to the z-score page, and verify the same list is still there.

## Cost

The server serves the page. The arithmetic and SVG rendering happen on the visitor's CPU. That removes the per-visit compute and storage cost of a server-side calculator, and it means there is no dataset to retain, protect or delete.

It does not make hosting free. The domain, the edge deployment, the fonts and the engineering time are real. The narrower claim is the useful one: there is no per-visit statistics-processing bill.

## Limits

- Courses disagree about quartiles; the tool exposes the disagreement instead of choosing silently.
- The 1.5 × IQR rule is a screening heuristic, not a significance test.
- The normal curve is a model, not a guarantee about the data.
- Dataset-mode z-scores use the sample SD (`n − 1`); a known population σ belongs in single-value mode.
- The browser tab is a finite machine; very large pastes are bounded by device memory.

This repository is a public showcase. The production application source is private.
