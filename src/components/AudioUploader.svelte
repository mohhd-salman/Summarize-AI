<script>
    const API_BASE_URL = process.env.API_BASE_URL || "http://localhost:5000";
    import {onMount} from "svelte";
    import LoadingSpinner from "./LoadingSpinner.svelte";
    import {
        transcription,
        currentStep,
        isLoading,
        error,
        mode,
    } from "../stores/appStore";

    let audioFile = null;
    let audioName = "";
    let audioPreview = null;
    let isRecording = false;
    let mediaRecorder;
    let audioChunks = [];
    let dragActive = false;
    let recordedAudio = null;
    let audioElement;

    onMount(() => {
        transcription.set("");
        error.set(null);
    });

    // File Upload Handling
    function handleFileInput(e) {
        if (e.target.files && e.target.files[0]) {
            handleFile(e.target.files[0]);
        }
    }

    function handleFile(file) {
        if (!file.type.startsWith("audio/")) {
            error.set("Please upload a valid audio file.");
            return;
        }
        audioFile = file;
        audioName = file.name;
        error.set(null);
        const url = URL.createObjectURL(file);
        audioPreview = url;
    }

    async function uploadAudio() {
        if (!audioFile) {
            error.set("Please select an audio file.");
            return;
        }

        isLoading.set(true);
        error.set(null);

        try {
            const formData = new FormData();
            formData.append("audio", audioFile);

            const response = await fetch(`${API_BASE_URL}/transcribe`, {
                method: "POST",
                body: formData,
            });

            if (!response.ok) {
                throw new Error(`Server responded with ${response.status}`);
            }

            const data = await response.json();
            transcription.set(data.transcription);
            currentStep.set(2);
        } catch (err) {
            error.set(`Error: ${err.message}`);
        } finally {
            isLoading.set(false);
        }
    }

    function removeAudio() {
        audioFile = null;
        audioName = "";
        audioPreview = null;
        error.set(null);
    }

    async function startRecording() {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({audio: true});
            mediaRecorder = new MediaRecorder(stream);
            audioChunks = [];

            mediaRecorder.ondataavailable = (event) => {
                audioChunks.push(event.data);
            };

            mediaRecorder.onstop = () => {
                const blob = new Blob(audioChunks, {type: "audio/wav"});
                recordedAudio = URL.createObjectURL(blob);
            };

            mediaRecorder.start();
            isRecording = true;
        } catch (err) {
            error.set(`Microphone access failed: ${err.message}`);
        }
    }

    function stopRecording() {
        if (mediaRecorder && isRecording) {
            mediaRecorder.stop();
            isRecording = false;
        }
    }

    function clearRecording() {
        recordedAudio = null;
        audioChunks = [];
    }

    async function uploadAudioBlob(blob) {
        isLoading.set(true);
        error.set(null);

        try {
            const formData = new FormData();
            formData.append("audio", blob, "audio.wav");

            const response = await fetch(`${API_BASE_URL}/transcribe`, {
                method: "POST",
                body: formData,
            });

            if (!response.ok) {
                throw new Error(`Server responded with ${response.status}`);
            }

            const data = await response.json();
            transcription.set(data.transcription);
            currentStep.set(2);
        } catch (err) {
            error.set(`Error: ${err.message}`);
        } finally {
            isLoading.set(false);
        }
    }

    // Drag and Drop
    function handleDragOver(e) {
        e.preventDefault();
        dragActive = true;
    }

    function handleDragLeave() {
        dragActive = false;
    }

    function handleDrop(e) {
        e.preventDefault();
        dragActive = false;

        if (e.dataTransfer.files && e.dataTransfer.files[0]) {
            handleFile(e.dataTransfer.files[0]);
        }
    }
</script>

<div class="mode-switcher">
    <button on:click={() => mode.set("file")} class:active={$mode === "file"}>
        Upload File
    </button>
    <button on:click={() => mode.set("record")} class:active={$mode === "record"}>
        Live Recording
    </button>
</div>

{#if $error}
    <div class="error-message">{$error}</div>
{/if}

{#if $isLoading}
    <LoadingSpinner message="Processing audio..."/>
{:else if $mode === "file"}
    <div class="card">
        <p>Upload an audio file to transcribe its content</p>

        {#if !audioFile}
            <div
                    class="upload-area"
                    class:active={dragActive}
                    on:dragover={handleDragOver}
                    on:dragleave={handleDragLeave}
                    on:drop={handleDrop}
            >
                <p>Drag and drop your audio file here</p>
                <label class="file-input-label">
                    Browse Files
                    <input type="file" accept="audio/*" on:change={handleFileInput} style="display: none;"/>
                </label>
                <p class="file-types">Supported formats: MP3, WAV, M4A, FLAC</p>
            </div>
        {:else}
            <div class="audio-preview">
                <p class="file-name">{audioName}</p>
                <div class="audio-controls">
                    <audio controls src={audioPreview}></audio>
                    <div class="buttons">
                        <button on:click={uploadAudio}>Transcribe Audio</button>
                        <button on:click={removeAudio}>Remove Audio</button>
                    </div>
                </div>
            </div>
        {/if}
    </div>
{:else}
    {#if $mode === 'record'}
        <div class="card">
            <p>Record your voice to transcribe its content</p>

            <div class="recording-controls">
                {#if !isRecording && !recordedAudio}
                    <button class="record-btn" on:click={startRecording}>🎤 Start Recording</button>
                {:else if isRecording}
                    <button class="stop-btn" on:click={stopRecording}>⏹️ Stop Recording</button>
                {:else}
                    <audio bind:this={audioElement} controls src={recordedAudio}/>
                    <div class="buttons">
                        <button on:click={() => uploadAudioBlob(new Blob(audioChunks, { type: 'audio/wav' }))}>
                            Transcribe Audio
                        </button>
                        <button on:click={clearRecording}>Clear</button>
                    </div>
                {/if}
            </div>
        </div>
    {/if}
{/if}

<style>
    .mode-switcher button {
        padding: 10px 20px;
        margin-right: 10px;
        background-color: var(--primary-color);
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    }

    .mode-switcher button.active {
        background-color: var(--primary-hover);
    }

    .mode-switcher {
        margin-bottom: 20px;
    }

    p {
        color: var(--light-text);
        margin-bottom: 1.5rem;
    }

    .upload-area {
        border: 2px dashed var(--border-color);
        border-radius: 0.5rem;
        padding: 2.5rem;
        text-align: center;
        transition: all 0.2s;
        cursor: pointer;
    }

    .upload-area.active {
        border-color: var(--primary-color);
        background-color: rgba(79, 70, 229, 0.05);
    }

    .upload-area p {
        margin-bottom: 0.5rem;
    }

    .file-input-label {
        display: inline-block;
        background-color: var(--primary-color);
        color: white;
        padding: 0.5rem 1.5rem;
        border-radius: 0.375rem;
        font-weight: 500;
        cursor: pointer;
        transition: background-color 0.2s;
        margin-bottom: 1rem;
    }

    .file-input-label:hover {
        background-color: var(--primary-hover);
    }

    .file-types {
        font-size: 0.75rem;
        color: var(--light-text);
        margin-top: 1rem;
    }

    .audio-preview {
        margin-bottom: 1.5rem;
    }

    .file-name {
        font-weight: 500;
        margin-bottom: 0.25rem;
    }

    .error-message {
        background-color: #fee2e2;
        color: var(--danger-color);
        padding: 0.75rem;
        border-radius: 0.375rem;
        margin-bottom: 1.5rem;
        font-size: 0.875rem;
    }

    .audio-controls {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        gap: 0.75rem;
    }

    .audio-controls audio {
        width: 100%;
    }

    .buttons {
        display: flex;
        gap: 1rem;
    }

</style>
