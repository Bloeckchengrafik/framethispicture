<script lang="ts">
    import corner from "$lib/corner.jpg"
    import line from "$lib/line.jpg"
    import Image from "image-js";

    let file: File | null = null;
    $: disabled = !file;
    let imageUrl: string | null = null;

    async function drawImgToCanvasAt(img: string, canvas: HTMLCanvasElement, x: number, y: number, rotate: number = 0) {
        let jsImg = await Image.load(img)
        jsImg = jsImg.rotate(rotate)
        let imageBitmap = await createImageBitmap(jsImg.getCanvas())
        let ctx = canvas.getContext("2d")!
        ctx.drawImage(imageBitmap, x, y)
    }

    async function frame() {
        if (!file) return;

        let img = await Image.load(await file.arrayBuffer())
        let cornerImg = await Image.load(corner)
        let lineImg = await Image.load(line)

        let cornerWidth = cornerImg.width
        let cornerHeight = cornerImg.height
        let lineHeight = lineImg.height

        let width = img.width + cornerWidth * 2
        let height = img.height + cornerHeight * 2

        /*
         * generate a frame around the image (corners and lines)
         */

        // create a canvas to draw the frame on
        let canvas = document.createElement("canvas")
        canvas.width = width
        canvas.height = height

        // draw the image (using file)
        await drawImgToCanvasAt(URL.createObjectURL(file), canvas, cornerWidth, cornerHeight)

        // draw the lines (repeat the line the first row and column)
        for (let x = 0; x < width; x += lineImg.width) {
            await drawImgToCanvasAt(line, canvas, x, 0)
            await drawImgToCanvasAt(line, canvas, x, height - lineHeight, 180)
        }

        for (let y = 0; y < height; y += lineImg.height) {
            await drawImgToCanvasAt(line, canvas, 0, y, 270)
            await drawImgToCanvasAt(line, canvas, width - lineHeight, y, 90)
        }

        // draw the corners
        await drawImgToCanvasAt(corner, canvas, 0, 0, 270)
        await drawImgToCanvasAt(corner, canvas, width - cornerWidth, 0, 0)
        await drawImgToCanvasAt(corner, canvas, width - cornerWidth, height - cornerHeight, 90)
        await drawImgToCanvasAt(corner, canvas, 0, height - cornerHeight, 180)

        // create a blob url for the file
        imageUrl = canvas.toDataURL()
    }
</script>

<svelte:head>
    <title>Frame This Picture</title>
</svelte:head>

<div class="w-[100vw] h-[100vh] flex items-center justify-center flex-col">
    {#if !imageUrl}
        <h1 class="text-4xl font-bold mb-4 tracking-tight">Frame This Picture</h1>
        <div class="join">
            <input type="file" class="join-item file-input file-input-bordered w-full max-w-xs" accept="image/*"
                   on:change={e => file = e.target.files[0]}/>
            <button class="join-item btn btn-primary" disabled={disabled} on:click={frame}>Frame It!</button>
        </div>
    {:else}
        <div class="flex flex-col items-center justify-center">
            <img src={imageUrl} alt="Your framed thing" class="max-h-[70vh]"/>
            <div class="join">
                <a href={imageUrl} download="framed.png" class="join-item btn btn-primary mt-4">Download</a>
                <button class="join-item btn btn-outline mt-4" on:click={() => imageUrl = null}>
                    Frame another picture
                </button>
            </div>
        </div>
    {/if}
</div>

<div class="bg-base-300 text-center p-3">
    &copy; 2023 <a href="https://bloeckchengrafik.de">Christian Bergschneider</a>. <a
        href="https://github.com/Bloeckchengrafik/framethispicture">Source Code</a> licensed under <a
        href="https://opensource.org/licenses/MIT">MIT</a>. Hosted on <a href="https://vercel.com">Vercel</a>. <a
        href="https://vercel.com">Vercel</a> is a registered trademark of Vercel, Inc. <a
        href="/datenschutz">Privacy</a>
</div>

<style lang="postcss">
    a {
        @apply underline;
    }
</style>