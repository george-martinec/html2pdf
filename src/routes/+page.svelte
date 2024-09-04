<script lang="ts">
    import { jsPDF } from "jspdf";
    import { onMount } from "svelte";

    let id = 1;
    let status: null|string;
    let html2pdf: any;
    let output: jsPDF;

    onMount(async () => {
        html2pdf = (await import('html2pdf.js')).default
        await load(1);
    });

    async function load(_id: number) {
        id = _id;
        status = 'loading ...';
        const preview = document.getElementById("preview") as HTMLIFrameElement;
        preview.src = '';

        const options = {
            margin: 0,
            enableLinks: true,
            image: {
                type: "jpeg",
                quality: 1,
            },
            html2canvas: {
                scale: 4,
                letterRendering: true,
                backgroundColor: null,
                useCORS: true,
            },
            pagebreak: {
                mode: [
                    "avoid-all",
                    "css",
                    "legacy",
                ],
            },
            jsPDF: {
                unit: "px",
                format: [794, 1123], // A4
                orientation: "portrait",
                compressPDF: true,
            },
        };

        const load = document.createElement('iframe') as HTMLIFrameElement
        document.body.appendChild(load);
        load.classList.add('w-[0px]', 'h-[0px]');

        load.src = "/pdf/" + id;
        load.onload = async () => {
            const loadWindow = load.contentWindow as Window;

            const header = loadWindow.document.body.getElementsByTagName("header")[0];
            const main = loadWindow.document.body.getElementsByTagName("main")[0];
            const footer = loadWindow.document.body.getElementsByTagName("footer")[0];

            await html2pdf()
                .set({
                    ...options,
                    margin: [
                        Math.round(header.getBoundingClientRect().height),
                        0,
                        Math.round(footer.getBoundingClientRect().height),
                        0,
                    ],
                })
                .from(main)
                .toPdf()
                .get("pdf")
                .then(async function (pdf: jsPDF) {
                    const theadImage = await html2pdf().set(options).from(header).outputImg("datauristring");
                    const tfootImage = await html2pdf().set(options).from(footer).outputImg("datauristring");

                    for (let page = 1; page < pdf.internal.pages.length; page++) {
                        const width = Math.round(pdf.internal.pageSize.width);
                        const height = Math.round(pdf.internal.pageSize.height);
                        pdf.setPage(page);

                        pdf.addImage(
                            theadImage,
                            'PNG',
                            0,
                            0,
                            Math.round(header.getBoundingClientRect().width),
                            Math.round(header.getBoundingClientRect().height),
                        );

                        pdf.addImage(
                            tfootImage,
                            'PNG',
                            0,
                            height - Math.round(header.getBoundingClientRect().height),
                            Math.round(header.getBoundingClientRect().width),
                            Math.round(header.getBoundingClientRect().height)
                        );
                    }

                    preview.src = URL.createObjectURL(pdf.output("blob"));
                    output = pdf;
                    status = 'download';
                });
            load.remove();
        }
    }

    async function download() {
        output.save('file.' + id + '.pdf');
    }
</script>

<div class="flex h-full gap-8 p-8 bg-slate-100">
    <div class="w-1/4 bg-white shadow-xl rounded-3xl overflow-hidden p-8">
        <button class="button button-blue" on:click={async () => await load(1)}>load 1</button>
        <button class="button button-blue" on:click={async () => await load(2)}>load 2</button>
        <div>
            <a href="/pdf/1"><button class="button button-green">open 1</button></a>
            <a href="/pdf/2"><button class="button button-green">open 2</button></a>
        </div>
        <div>
            {#if status}
                <button class="button button-red" on:click={async () => download()}>{status}</button>
            {/if}
        </div>
    </div>

    <div class="w-3/4 bg-white shadow-xl rounded-3xl overflow-hidden">
        <div class="w-full h-full rounded-3xl overflow-hidden">
            <iframe title="preview" id="preview" class="w-full h-full"/>
        </div>
    </div>
</div>