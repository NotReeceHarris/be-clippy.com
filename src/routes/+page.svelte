
<script lang="ts">

    import ColorPicker from 'svelte-awesome-color-picker';
    import { browser } from '$app/environment';
    import { onMount } from 'svelte';
    import clippys, { clippyCount } from '$lib/clippys';

    let watermark = $state(false)
    let hex = $state("#ffffff")
    let retry: number | null = $state(null)
    let noMouse = $state(false)

    let clippyCanvas: (HTMLCanvasElement | null)[] = $state(Array.from({ length: clippyCount }, (_) => null));
    let clippyImages: (HTMLImageElement | null)[] = $state(Array.from({ length: clippyCount }, (_) => null));
    let clippyLoading: boolean[] = $state(Array.from({ length: clippyCount }, (_) => true));

    // return either black or white based on background colour
    function getTextColour() {
        const h = hex.replace('#', '');
        const r = parseInt(h.substring(0, 2), 16);
        const g = parseInt(h.substring(2, 4), 16);
        const b = parseInt(h.substring(4, 6), 16);
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

    async function renderClippys() {
        if (!browser) return;
        if (retry) clearInterval(retry);

        Promise.all(
            clippys.map(async (_, i) => {
                const clippy = clippys[i];
                let canvas = clippyCanvas[i];

                if (!canvas) {
                    canvas = document.createElement('canvas');
                    clippyCanvas[i] = canvas;
                }

                canvas.width = 243;
                canvas.height = 243;
                const ctx = canvas.getContext('2d');
                if (!ctx) return;

                // draw background
                ctx.fillStyle = hex;
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
                    await ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    return clippyLoading[i] = false;
                }

                img = new Image();
                img.src = '/clippys/' + clippy.image;
                clippyImages[i] = img;

                img.onload = async () => {
                    await ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    clippyLoading[i] = false;
                };
            })
        )

    }

    onMount(()=>{
        if (!browser) return;
        retry = setInterval(renderClippys, 1000);

        if(window.matchMedia("(any-hover: none)").matches) {
            noMouse = true;
        }
    })

</script>

<div>

    <div class="mx-auto max-w-screen-md px-6 md:px-0 h-full w-full py-8 flex flex-col gap-6">

        <div class="flex flex-col gap-5 text-xl text-black/70">
            <h1 class="text-4xl font-bold text-black">
                Be Like Clippy 📎
            </h1>

            <strong>
                "Clippy didn’t sell your data. Clippy didn’t hold your data hostage. Clippy was there to help you."
            </strong>

            <p>
                Fed up with trillion-dollar companies exploiting your data?
                Forced to use their services?
                Your data held for ransom?
                Your data used to train their AI models?
                Opt-outs for data collection instead of opt-ins?
            </p>

            <p>
                Join the movement to make companies more like Clippy. <span class="text-black/90 font-semibold">Set your profile picture to Clippy</span>, make your voice heard. 
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

            <ColorPicker
                bind:hex
                onInput={renderClippys}
                label="Background color"
                position="responsive"
            />
        </div>

        <div class="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">

            {#each clippys as _, clippyIndex}
                <div class="group relative flex flex-col place-items-center place-content-center aspect-square rounded-lg border border-black/30">
                    <button onclick={()=>download(clippyIndex)} class="{noMouse ? '' : 'group-hover:block md:hidden'} absolute top-2 right-2 rounded-lg bg-white border text-black/90 border-black/30 p-2 cursor-pointer" aria-label="Download">
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" viewBox="0 0 256 256"><path opacity="1" d="M224,144v64a8,8,0,0,1-8,8H40a8,8,0,0,1-8-8V144a8,8,0,0,1,16,0v56H208V144a8,8,0,0,1,16,0Zm-101.66,5.66a8,8,0,0,0,11.32,0l40-40a8,8,0,0,0-11.32-11.32L136,124.69V32a8,8,0,0,0-16,0v92.69L93.66,98.34a8,8,0,0,0-11.32,11.32Z"></path></svg>
                    </button>
                    {#if clippyLoading[clippyIndex]}
                        <svg class="z-10 absolute size-8 animate-spin opacity-70" style="color: {getTextColour()};" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                    {/if}

                    <canvas bind:this={clippyCanvas[clippyIndex]} class="aspect-square p-8 md:p-0 w-full rounded-lg"></canvas>
                </div>
            {/each}

        </div>

        <footer class="flex flex-col sm:flex-row place-items-center gap-3 text-black/70 justify-between">

            <p>License under GPL-3.0</p>

            <p>
                Share your own on <a class="underline" href="https://github.com/NotReeceHarris/be-clippy.com">Github</a>.
            </p>

        </footer>
    
    </div>

</div>