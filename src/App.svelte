<script>
    import { writable } from 'svelte/store';

    const inputPrompt = writable('');
    const imageUrl = writable('');
    const isLoading = writable(false);

    async function generateImage() {
        if ($isLoading) {
            // Optionally, inform the user or simply return to avoid concurrent generations
            console.log("Please wait for the current image to finish generating.");
            return;
        }

        isLoading.set(true);
        imageUrl.set(''); // Optionally reset the image URL if you want to clear the previous image immediately

        const url = "https://api.runpod.ai/v2/vps0rnvp1ud2uy/runsync"; // Your API URL
        const headers = {
            "Accept": "application/json",
            "Content-Type": "application/json",
            "Authorization": "Bearer ${import.meta.env.VITE_API_TOKEN}",
        };
        const data = {
            "input": {
                "prompt": $inputPrompt,
                "num_inference_steps": 9,
                "width": 1024,
                "height": 1024,
                "guidance_scale": 3,
                "seed": null, // Consider updating or removing the seed for new random results
                "num_images": 1,
            }
        };

        try {
            const response = await fetch(url, {
                method: 'POST',
                headers: headers,
                body: JSON.stringify(data)
            });

            const responseData = await response.json();
            if (response.ok && responseData.output) {
                const base64Image = responseData.output.split(",")[1];
                imageUrl.set(`data:image/png;base64,${base64Image}`);
            } if (response.)
			else {
                throw new Error(responseData.message || 'Failed to generate image');
            }
        } catch (error) {
            console.error('Error generating image:', error);
        } finally {
            isLoading.set(false);
        }
    }
</script>

<div class="container">
    <h1>Generate your images</h1>
    <input type="text" bind:value={$inputPrompt} placeholder="Enter your prompt" />
    <button on:click={generateImage} disabled={$isLoading}>{$isLoading ? 'Generating...' : 'Generate Image'}</button>

    {#if $imageUrl}
        <img src={$imageUrl} alt="Generated Image" />
    {/if}
</div>
<style>
    .container {
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;
        height: 100vh;
        padding-top: 20px;
    }
	h1 {
        margin: 0 0 20px;
        text-align: center;
        font-size: 24px;
    }
    button {
        margin-top: 10px;
    }

	img {
    margin-top: 20px;
    max-width: 100%;
    height: auto;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

</style>