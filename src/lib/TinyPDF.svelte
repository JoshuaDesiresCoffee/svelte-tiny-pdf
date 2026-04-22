<script>
    import { onMount } from "svelte";
    import { GlobalWorkerOptions, getDocument, version as pdfjsVersion } from "pdfjs-dist";
    import "pdfjs-dist/web/pdf_viewer.css";
    import * as pdfjs from "pdfjs-dist/web/pdf_viewer.mjs";

    let {
        config = {
            url: "",
            data: null,
            scale: {
                initial: 1,
                step: 0.2,
                min: 0.4,
                max: 2
            },
            innerscroll: true,
            removeBorders: false,
            style: "",
            workerSrc: "",
            workerPort: null
        }
    } = $props();

    let container;
    let pdfDocument;
    let viewer;

    const configureWorker = () => {
        if (config.workerPort) {
            GlobalWorkerOptions.workerPort = config.workerPort;
            return;
        }
        GlobalWorkerOptions.workerSrc =
            config.workerSrc ||
            `https://cdn.jsdelivr.net/npm/pdfjs-dist@${pdfjsVersion}/build/pdf.worker.min.mjs`;
    };

    const init = () => {
        const eventBus = new pdfjs.EventBus();
        const linkService = new pdfjs.PDFLinkService({
            eventBus,
            externalLinkTarget: 2
        });

        viewer = new pdfjs.PDFViewer({
            container,
            eventBus,
            textLayerMode: 2,
            removePageBorders: config.removeBorders,
            linkService
        });

        linkService.setDocument(pdfDocument);
        linkService.setViewer(viewer);
        viewer.setDocument(pdfDocument);
    };

    const loadDocument = async () => {
        if (!container) return;
        configureWorker();

        const loadingTask = config.url
            ? getDocument({ url: config.url })
            : getDocument({ data: config.data });

        pdfDocument = await loadingTask.promise;
        init();
    };

    onMount(() => {
        loadDocument().then(() => {
            if (viewer && config.scale?.initial) {
                viewer.currentScale = config.scale.initial;
            }
        });

        return () => {
            if (pdfDocument) {
                pdfDocument.destroy();
            }
        };
    });

    export function getCurrentPage() {
        return viewer.currentPageNumber;
    }

    export function getPageCount() {
        return pdfDocument.numPages;
    }

    export async function getPageText(pageNumber) {
        const page = await pdfDocument.getPage(pageNumber);
        const textContent = await page.getTextContent();
        return textContent.items.map((item) => item.str).join(" ");
    }

    export function scaleUp() {
        if (viewer.currentScale + config.scale.step < config.scale.max) {
            viewer.currentScale += config.scale.step;
        }
    }

    export function scaleDown() {
        if (viewer.currentScale - config.scale.step > config.scale.min) {
            viewer.currentScale -= config.scale.step;
        }
    }

    export function fitWidth() {
        viewer.currentScaleValue = "page-width";
    }

    export function fitHeight() {
        viewer.currentScaleValue = "page-height";
    }

    export function jumpPage(pageNumber) {
        viewer.scrollPageIntoView({ pageNumber });
    }

    export function jumpPageNext() {
        if (viewer.currentPageNumber < pdfDocument.numPages) {
            viewer.currentPageNumber += 1;
        }
    }

    export function jumpPagePrevious() {
        if (viewer.currentPageNumber > 1) {
            viewer.currentPageNumber -= 1;
        }
    }

    export async function download(filename) {
        const resp = await fetch(config.url, { method: "GET" });
        const blob = await resp.blob();
        const url = window.URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.style.display = "none";
        a.href = url;
        a.download = filename ?? "file.pdf";
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        window.URL.revokeObjectURL(url);
    }
</script>

<div
    bind:this={container}
    class="pdfViewer"
    class:innerscroll={config.innerscroll}
    style={config.style}
>
    <div></div>
</div>

<style>
    .pdfViewer {
        width: 100%;
        position: absolute;
    }
    .innerscroll {
        height: 100%;
        overflow: auto;
    }
</style>
