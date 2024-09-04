<script lang="ts">
    import { page } from '$app/stores';
    const id = $page.params.id;
</script>

<style>
    page {
        margin: auto;
        display: block;
        width: 794px; /* A4 */
        height: 1123px; /* A4 */
    }
</style>

<page>
    <header>
        <div class="w-20 h-20 bg-blue-500">header</div>
        <div class="w-full h-20 bg-amber-500 border border-red-500">line</div>
    </header>

    <main>
        <!-- When using images define width and height, the image should be as larger as the scale of the PDF, in our case it is scale by html2canvas.scale: 4 -->
        <img src="https://picsum.photos/400/800" width="100" height="200" />
        {#each {length: 6} as _, i}
            <div class="w-20 h-20 bg-gray-500">main</div>
            <div class="w-full h-[200px] bg-green-500 border border-red-500">{id} {i + 1}</div>
        {/each}
    </main>

    <footer>
        <div class="w-20 h-20 bg-red-500">footer</div>
        <div class="w-full h-20 bg-purple-500 border border-red-500">line</div>
    </footer>
</page>