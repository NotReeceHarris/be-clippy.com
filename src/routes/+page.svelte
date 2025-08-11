
<script lang="ts">

    import { browser } from '$app/environment';
    import { onMount } from 'svelte';
    import clippys from '$lib/clippys';

    let watermark = $state(false)
    let background = $state("#ffffff")

    let clippyCanvas: (HTMLCanvasElement | null)[] = Array.from({ length: clippys.length }, (_, i) => null);
    let clippyImages: (HTMLImageElement | null)[] = Array.from({ length: clippys.length }, (_, i) => null);

    // return either black or white based on background colour
    function getTextColour() {
        const hex = background.replace('#', '');
        const r = parseInt(hex.substring(0, 2), 16);
        const g = parseInt(hex.substring(2, 4), 16);
        const b = parseInt(hex.substring(4, 6), 16);
        const brightness = (r * 299 + g * 587 + b * 114) / 1000;
        return brightness > 128 ? 'black' : 'white';
    }

    function download(clippy: number) {
        const canvas = clippyCanvas[clippy];
        if (!canvas) return;

        const link = document.createElement('a');
        link.download = clippys[clippy].name + '.png';
        link.href = canvas.toDataURL();
        link.click();
        link.remove();
    }

    function renderClippys() {
        if (!browser) return;

        for (let i = 0; i < clippys.length; i++) {
            try {
                const clippy = clippys[i];
                let canvas = clippyCanvas[i];

                if (!canvas) {
                    canvas = document.createElement('canvas');
                    clippyCanvas[i] = canvas;
                }

                canvas.width = 243;
                canvas.height = 243;
                const ctx = canvas.getContext('2d');
                if (!ctx) continue;


                // draw background
                ctx.fillStyle = background;
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                // draw text to canvas "be-clippy.com" rotated 90 degress
                if (watermark) {
                    ctx.save();
                    ctx.font = "16px sans-serif";
                    ctx.fillStyle = getTextColour();
                    ctx.translate((canvas.width / 2) + 40, (canvas.height / 2) - 10);
                    ctx.rotate(Math.PI / 2);
                    ctx.fillText("be-clippy.com", 0, 0);
                    ctx.restore();
                }

                let img = clippyImages[i];

                if (img) {
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    continue;
                }

                img = new Image();
                img.src = '/clippys/' + clippy.image;
                clippyImages[i] = img;

                img.onload = () => {
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                };
            } catch (error) {
                console.error(error);
            }

        }

    }

    onMount(renderClippys)

</script>

<div>

    <div class="mx-auto max-w-screen-md h-full w-full py-8 flex flex-col gap-6">

        <div class="flex flex-col gap-5 text-xl text-black/70">
            <h1 class="text-4xl font-bold text-black">
                Be Like Clippy 📎
            </h1>

            <strong>
                "Clippy didn’t sell your data. Clippy didn’t hold your data hostage. Clippy was there to help you."
            </strong>

            <p>
                Are you fed up with trillion-dollar companies exploiting your data? Forcing you to use their services? Holding your data for ransom? Using your data to train their AI models? Using opt-outs for data collection instead of opt-ins?
            </p>

            <p>
                Join the movement to make companies more like Clippy. <span class="text-black/90 font-semibold">Set your profile picture to a Clippy</span>, make your voice heard. 
            </p>

            <p>
                Below is a video that explains the Be Like Clippy movement. It’s a call to action for developers, companies, and users alike to embrace a more open, transparent, and user-friendly approach to technology.
            </p>
        </div>

        <iframe class="rounded-lg aspect-video w-full" src="https://www.youtube-nocookie.com/embed/2_Dtmpe9qaQ?si=A9g2Ufxt_8vzXW4p" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
        
        <div class="flex gap-8 place-items-center">
            <label for="watermark">
                <input type="checkbox" id="watermark" bind:checked={watermark} onchange={renderClippys}>
                Link movement on your profile picture.
            </label>

            <div class="flex place-items-center gap-2">
                <input type="color" id="background" bind:value={background} onchange={renderClippys}>
                <label for="background">
                    Background color
                </label>
            </div>
        </div>

        <div class="grid grid-cols-3 gap-4">

            {#each clippys as clippy, clippyIndex}
                <div class="group relative flex flex-col place-items-center place-content-center aspect-square rounded-lg border border-black/30">
                    <button onclick={()=>download(clippyIndex)} class="group-hover:block hidden absolute top-2 right-2 rounded-lg bg-white border text-black/90 border-black/30 p-2 cursor-pointer" aria-label="Download">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 256 256"><path opacity="1" d="M224,144v64a8,8,0,0,1-8,8H40a8,8,0,0,1-8-8V144a8,8,0,0,1,16,0v56H208V144a8,8,0,0,1,16,0Zm-101.66,5.66a8,8,0,0,0,11.32,0l40-40a8,8,0,0,0-11.32-11.32L136,124.69V32a8,8,0,0,0-16,0v92.69L93.66,98.34a8,8,0,0,0-11.32,11.32Z"></path></svg>
                    </button>
                    <canvas bind:this={clippyCanvas[clippyIndex]} class="rounded-lg"></canvas>
                </div>
            {/each}

        </div>
    
    </div>

</div>