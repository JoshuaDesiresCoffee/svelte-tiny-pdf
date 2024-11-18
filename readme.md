# svelte-tiny-pdf

A lightweight PDF viewer component for Svelte using PDF.js.

## Installation

```bash
npm install svelte-tiny-pdf
```

## Usage

```svelte
<script>
  import { TinyPDF } from 'svelte-tiny-pdf';
  
  let pdfUrl = 'path/to/your/document.pdf';
</script>

<TinyPDF config={{url: pdfUrl}} />
```

## Configuration Options

- `url`: Path to PDF file (string)
- `data`: PDF data (optional)
- `scale`: Zoom configuration
  - `initial`: Initial zoom level (default: 1)
  - `step`: Zoom increment (default: 0.2)
  - `min`: Minimum zoom (default: 0.4)
  - `max`: Maximum zoom (default: 2)
- `innerscroll`: Enable inner scrolling (default: true)
- `style`: Additional CSS styles

## Methods

- `getCurrentPage()`
- `getPageCount()`
- `getPageText(pageNumber)`
- `scaleUp()`
- `scaleDown()`
- `fitWidth()`
- `fitHeight()`
- `jumpPage(pageNumber)`
- `jumpPageNext()`
- `jumpPagePrevious()`
- `download(filename)`

## License

MIT