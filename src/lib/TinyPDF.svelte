<script>
    import { onMount } from "svelte";
    import { GlobalWorkerOptions, getDocument } from "pdfjs-dist";
    import "pdfjs-dist/web/pdf_viewer.css";
    import workerSrc from "pdfjs-dist/build/pdf.worker.mjs?url";
    import * as pdfjs from "pdfjs-dist/web/pdf_viewer.mjs";

    let container;
    let pdfDocument;
    let viewer;

    export let config = {
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
        style: ""
    }

    const init = async () => {
        let eventBus = new pdfjs.EventBus();
        const linkService = new pdfjs.PDFLinkService({
            eventBus,
            externalLinkTarget: 2,
        });

        viewer = new pdfjs.PDFViewer({
            container,
            eventBus,
            textLayerMode: 2,
            removePageBorders: config.showBorder,
            linkService,
        });

        linkService.setDocument(pdfDocument);
        linkService.setViewer(viewer);
        viewer.setDocument(pdfDocument);
    };

    const loadDocument = async () => {
        if(!container) return;
        GlobalWorkerOptions.workerSrc = workerSrc;

        let loadingTask;
        if(config.url !== "") {
            loadingTask = getDocument({ url: config.url });
        }
        else {
            loadingTask = getDocument({ data: config.data });
        }

        pdfDocument = await loadingTask.promise;
        init();
    };

    onMount(() => {
        loadDocument().then(() => {
            // Causes weird error on load, but is unrelated
            viewer.currentScale = config.scale.initial;
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
        const textItems = textContent.items.map(item => item.str);
        return textItems.join(" ");
    }

    export function scaleUp() {
        if(viewer.currentScale + config.scale.step < config.scale.max) {
            viewer.currentScale += config.scale.step;
        }
    }

    export function scaleDown() {
        if(viewer.currentScale - config.scale.step > config.scale.min) {
            viewer.currentScale -= config.scale.step;
        }
    }

    export function fitWidth() {
        viewer.currentScaleValue = 'page-width';
    }

    export function fitHeight() {
        viewer.currentScaleValue = 'page-height';
    }

    export function jumpPage(pageNumber) {
        viewer.scrollPageIntoView({pageNumber});
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

    export function download(filename) {
        fetch(config.url, {
            method: 'GET'
        })
        .then(resp => resp.blob())
        .then(blob => {
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.style.display = 'none';
            a.href = url;
            a.download = filename ?? "file.pdf";
            document.body.appendChild(a);
            a.click();
            window.URL.revokeObjectURL(url);
        })
    }
</script>

<div bind:this={container} class={`pdfViewer ${config.innerscroll && "innerscroll"}`} style={config.style}>
    <div></div>
</div>

<style>
    .pdfViewer {
        width: 100%;
        position: absolute;
    }
    .innerscroll {
        height: 100%; /* TODO: change to 100% */
        overflow: auto;
    }
</style>
