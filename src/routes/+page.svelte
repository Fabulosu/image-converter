<script lang="ts">
    import { fade, fly } from 'svelte/transition';
    
    let selectedFile: File | null = null;
    let format: 'webp' | 'jpg' | 'png' | 'gif' | 'bmp' | 'tiff' | 'ico' | 'avif' = 'webp';
    let convertedUrl: string = '';
    let isDragging = false;
    let isLoading = false;
    let fileInput: HTMLInputElement;

    const handleSubmit = async (): Promise<void> => {
        if (!selectedFile) {
            alert('Please select a file before submitting');
            return;
        }

        isLoading = true;
        const formData = new FormData();
        formData.append('image', selectedFile);
        formData.append('format', format);

        try {
            const response = await fetch('https://image-converter-backend-production.up.railway.app/api/convert', {
                method: 'POST',
                body: formData
            });

            if (!response.ok) throw new Error('Conversion failed');

            const blob = await response.blob();
            convertedUrl = URL.createObjectURL(blob);
        } catch (error) {
            alert(error instanceof Error ? error.message : 'Unknown error occurred');
        } finally {
            isLoading = false;
        }
    };

    const handleDragEnter = (e: DragEvent) => {
        e.preventDefault();
        isDragging = true;
    };

    const handleDragLeave = (e: DragEvent) => {
        e.preventDefault();
        isDragging = false;
    };

    const handleDrop = (e: DragEvent) => {
        e.preventDefault();
        isDragging = false;
        const files = e.dataTransfer?.files;
        if (files?.[0]) selectedFile = files[0];
    };

    const handleZoneClick = () => {
        fileInput.click();
    };
</script>
<title>Online Image Converter</title>
<div 
    class="min-h-screen bg-gray-900 p-4 sm:p-8 flex flex-col justify-center items-center"
    in:fade
>
    <div class="w-full max-w-xl bg-gray-800 rounded-2xl shadow-2xl p-6 sm:p-8 border border-gray-700">
        <h1 class="text-3xl font-bold text-center text-blue-400 mb-8 tracking-tight">
            Image Converter
        </h1>

        <form 
            on:submit|preventDefault={handleSubmit}
            class="space-y-6"
        >
            <!-- Drop Zone -->
            <!-- svelte-ignore a11y_click_events_have_key_events -->
            <div
                class={`
                    border-2 border-dashed rounded-xl p-8 sm:p-12
                    text-center cursor-pointer transition-all duration-300 ease-in-out
                    ${isDragging 
                        ? 'border-blue-400 bg-blue-500/10' 
                        : 'border-gray-600 hover:border-blue-500 hover:bg-gray-700/50'}
                `}
                on:dragenter={handleDragEnter}
                on:dragover|preventDefault
                on:dragleave={handleDragLeave}
                on:drop={handleDrop}
                on:click={handleZoneClick}
                role="button"
                tabindex="0"
            >
                <input
                    bind:this={fileInput}
                    type="file"
                    accept="image/*"
                    class="hidden"
                    on:change={(e) => (selectedFile = (e.target as HTMLInputElement).files?.[0] || null)}
                    required
                />
                
                {#if selectedFile}
                    <div class="space-y-2">
                        <svg class="w-8 h-8 mx-auto text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                        </svg>
                        <p class="text-gray-300">
                            Selected: <span class="font-medium text-blue-400">{selectedFile.name}</span>
                        </p>
                        <p class="text-sm text-gray-500">Click or drag to change file</p>
                    </div>
                {:else}
                    <div class="space-y-4">
                        <svg class="w-12 h-12 mx-auto text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
                        </svg>
                        <div>
                            <p class="text-gray-300 font-medium">
                                Drop your image here
                            </p>
                            <p class="text-sm text-gray-500 mt-1">
                                or click to browse
                            </p>
                        </div>
                        <p class="text-xs text-gray-500">
                            Supports: JPG, PNG, WebP, and more
                        </p>
                    </div>
                {/if}
            </div>

            <!-- Format Selection -->
            <select
                bind:value={format}
                class="w-full px-4 py-3 bg-gray-700 border border-gray-600 rounded-lg
                    text-gray-200 cursor-pointer appearance-none
                    focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500
                    transition-colors duration-200"
            >
                <option value="webp">Convert to WebP</option>
                <option value="jpg">Convert to JPG</option>
                <option value="png">Convert to PNG</option>
                <option value="gif">Convert to GIF</option>
                <option value="bmp">Convert to BMP</option>
                <option value="tiff">Convert to TIFF</option>
                <option value="ico">Convert to ICO</option>
                <option value="avif">Convert to AVIF</option>
            </select>

            <!-- Convert Button -->
            <button
                type="submit"
                disabled={!selectedFile || isLoading}
                class="w-full px-4 py-3 text-white font-medium rounded-lg
                       transition-all duration-200 transform
                       {isLoading || !selectedFile 
                           ? 'bg-gray-600 cursor-not-allowed' 
                           : 'bg-blue-600 hover:bg-blue-700 hover:scale-[1.02] active:scale-[0.98]'}"
            >
                {#if isLoading}
                    <div class="flex items-center justify-center">
                        <div class="w-5 h-5 border-2 border-white border-t-transparent rounded-full animate-spin-custom"></div>
                    </div>
                {:else}
                    Convert Image
                {/if}
            </button>
        </form>

        <!-- Download Section -->
        {#if convertedUrl}
            <div 
                class="mt-8 text-center space-y-4"
                in:fly={{ y: 20, duration: 300 }}
            >
                <p class="text-gray-300">Your image is ready!</p>
                <a
                    href={convertedUrl}
                    download="converted-image"
                    class="inline-block px-6 py-3 bg-green-600 text-white font-medium rounded-lg
                           hover:bg-green-700 transition-all duration-200 transform hover:scale-[1.02]
                           active:scale-[0.98]"
                >
                    Download Converted Image
                </a>
            </div>
        {/if}
    </div>

    <footer class="mt-8 text-center text-gray-500 text-sm space-y-2">
        <p>Convert your images to any format, instantly.</p>
        <p class="text-gray-400">
            Made with <span class="text-red-500">❤️</span> by 
            <a 
                href="https://fabulosu.xyz" 
                target="_blank" 
                rel="noopener noreferrer" 
                class="text-blue-400 hover:text-blue-300 transition-colors duration-200 font-medium"
            >
                Fabulosu
            </a>
        </p>
    </footer>
</div>

<style>
    @keyframes spin {
        to {
            transform: rotate(360deg);
        }
    }

    .animate-spin-custom {
        animation: spin 1s linear infinite;
    }

    /* Custom scrollbar for dark theme */
    :global(::-webkit-scrollbar) {
        width: 8px;
    }

    :global(::-webkit-scrollbar-track) {
        background: #1f2937;
    }

    :global(::-webkit-scrollbar-thumb) {
        background: #4b5563;
        border-radius: 4px;
    }

    :global(::-webkit-scrollbar-thumb:hover) {
        background: #6b7280;
    }
</style>