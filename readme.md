# svelte-tiny-pdf

A lightweight PDF viewer component for Svelte 5 using PDF.js.

## Installation

```bash
npm install svelte-tiny-pdf
```

Requires Svelte 5. `pdfjs-dist` is a dependency and is installed automatically.

## Usage

```svelte
<script>
  import { TinyPDF } from 'svelte-tiny-pdf';

  let pdfUrl = 'path/to/your/document.pdf';
  let pdf;
</script>

<TinyPDF bind:this={pdf} config={{ url: pdfUrl }} />

<button onclick={() => pdf.jumpPageNext()}>Next page</button>
```

## The PDF worker

PDF.js runs its parser in a Web Worker, which must be loaded from its own URL — the browser can't run it from the main bundle. By default this component loads the worker from a CDN matching the installed `pdfjs-dist` version, so it works without any setup.

If you'd rather bundle the worker yourself (recommended for production, offline use, or strict CSPs), pass a `workerSrc` or `workerPort` via config:

**Vite / SvelteKit — static URL:**

```svelte
<script>
  import { TinyPDF } from 'svelte-tiny-pdf';
  import workerSrc from 'pdfjs-dist/build/pdf.worker.min.mjs?url';
</script>

<TinyPDF config={{ url: pdfUrl, workerSrc }} />
```

**Vite / SvelteKit — worker instance:**

```svelte
<script>
  import { TinyPDF } from 'svelte-tiny-pdf';
  import PdfWorker from 'pdfjs-dist/build/pdf.worker.min.mjs?worker';

  const workerPort = new PdfWorker();
</script>

<TinyPDF config={{ url: pdfUrl, workerPort }} />
```

## Configuration

Pass a `config` object as a prop. All fields are optional except `url` or `data`.

| Option          | Type              | Default      | Description                                                      |
| --------------- | ----------------- | ------------ | ---------------------------------------------------------------- |
| `url`           | `string`          | `""`         | URL to the PDF file.                                             |
| `data`          | `Uint8Array`      | `null`       | Raw PDF data. Used when `url` is empty.                          |
| `scale.initial` | `number`          | `1`          | Initial zoom level.                                              |
| `scale.step`    | `number`          | `0.2`        | Zoom step for `scaleUp` / `scaleDown`.                           |
| `scale.min`     | `number`          | `0.4`        | Minimum zoom.                                                    |
| `scale.max`     | `number`          | `2`          | Maximum zoom.                                                    |
| `innerscroll`   | `boolean`         | `true`       | Scroll inside the component instead of the page.                 |
| `removeBorders` | `boolean`         | `false`      | Hide page borders.                                               |
| `style`         | `string`          | `""`         | Extra inline CSS for the viewer container.                       |
| `workerSrc`     | `string`          | CDN URL      | Override the PDF.js worker URL.                                  |
| `workerPort`    | `Worker`          | `null`       | Provide a pre-constructed worker (takes precedence over `workerSrc`). |

## Methods

Bind the component with `bind:this` to call:

- `getCurrentPage()` — current page number
- `getPageCount()` — total page count
- `getPageText(pageNumber)` — extracted text from a page
- `scaleUp()` / `scaleDown()` — step the zoom
- `fitWidth()` / `fitHeight()` — fit the page
- `jumpPage(pageNumber)` — scroll to a specific page
- `jumpPageNext()` / `jumpPagePrevious()` — step through pages
- `download(filename)` — download the PDF (only when `url` is set)

## License

MIT
