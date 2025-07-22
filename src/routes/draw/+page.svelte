<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  let images: { url: string; file: File }[] = [];
  let fileInput: HTMLInputElement;
  let cameraInput: HTMLInputElement;

  // Camera modal state
  let showCamera = false;
  let videoEl: HTMLVideoElement;
  let stream: MediaStream | null = null;
  let canvasEl: HTMLCanvasElement;
  let availableCameras: MediaDeviceInfo[] = [];
  let selectedCameraId = '';

  function handleFiles(event: Event) {
    const files = (event.target as HTMLInputElement).files;
    if (!files) return;
    const filesArr = Array.from(files).slice(0, 3 - images.length);
    for (const file of filesArr) {
      const url = URL.createObjectURL(file);
      images = [...images, { url, file }];
    }
    fileInput.value = '';
  }

  function handleCamera(event: Event) {
    const files = (event.target as HTMLInputElement).files;
    if (!files) return;
    const file = files[0];
    if (file && images.length < 3) {
      const url = URL.createObjectURL(file);
      images = [...images, { url, file }];
    }
    cameraInput.value = '';
  }

  async function getAvailableCameras() {
    try {
      const devices = await navigator.mediaDevices.enumerateDevices();
      availableCameras = devices.filter(device => device.kind === 'videoinput');
      if (availableCameras.length > 0) {
        selectedCameraId = availableCameras[0].deviceId;
      }
    } catch (err) {
      console.error('Error getting cameras:', err);
    }
  }

  async function openCamera() {
    if (images.length < 3) {
      showCamera = true;
      await getAvailableCameras();
      await startCamera();
    }
  }

  async function startCamera() {
    try {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
      }
      
      const constraints = {
        video: selectedCameraId ? { deviceId: { exact: selectedCameraId } } : true
      };
      
      stream = await navigator.mediaDevices.getUserMedia(constraints);
      if (videoEl) {
        videoEl.srcObject = stream;
        await videoEl.play();
      }
    } catch (err) {
      alert('Could not access camera.');
      showCamera = false;
    }
  }

  async function switchCamera() {
    if (availableCameras.length > 1) {
      const currentIndex = availableCameras.findIndex(cam => cam.deviceId === selectedCameraId);
      const nextIndex = (currentIndex + 1) % availableCameras.length;
      selectedCameraId = availableCameras[nextIndex].deviceId;
      await startCamera();
    }
  }

  function closeCamera() {
    showCamera = false;
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      stream = null;
    }
  }

  async function capturePhoto() {
    if (!videoEl || !canvasEl) return;
    canvasEl.width = videoEl.videoWidth;
    canvasEl.height = videoEl.videoHeight;
    const ctx = canvasEl.getContext('2d');
    if (ctx) {
      ctx.drawImage(videoEl, 0, 0, canvasEl.width, canvasEl.height);
      canvasEl.toBlob(blob => {
        if (blob && images.length < 3) {
          const file = new File([blob], `photo-${Date.now()}.png`, { type: 'image/png' });
          const url = URL.createObjectURL(file);
          images = [...images, { url, file }];
          closeCamera();
        }
      }, 'image/png');
    }
  }

  function removeImage(idx: number) {
    URL.revokeObjectURL(images[idx].url);
    images = images.filter((_, i) => i !== idx);
  }

  function finalise() {
    alert('Images finalised!');
  }

  onDestroy(() => {
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
    }
  });
</script>

<div class="plex font-extralight text-4xl">
    welcome to the draw section
</div>

<a href="/" class="plex">back</a>

<div class="mt-6 flex flex-col gap-4 items-center">
    <p>Upload exactly 3 images of yourself</p>
    {#if images.length < 3}
      <input
        type="file"
        accept="image/*"
        multiple
        bind:this={fileInput}
        on:change={handleFiles}
        style="display: none;"
        disabled={images.length >= 3}
      />
      <input
        type="file"
        accept="image/*"
        capture="environment"
        bind:this={cameraInput}
        on:change={handleCamera}
        style="display: none;"
        disabled={images.length >= 3}
      />
      <div class="flex flex-col gap-2">
        <button class="plex" on:click={() => fileInput.click()} disabled={images.length >= 3}>
          Upload from device
        </button>
        <button class="plex" on:click={openCamera} disabled={images.length >= 3}>
          Take an image now
        </button>
      </div>
    {/if}
    <div class="flex gap-4 flex-wrap justify-center">
      {#each images as img, idx}
        <div class="relative inline-block">
          <img src={img.url} alt="user photos" class="w-32 h-32 object-cover rounded shadow" />
          <button
            class="absolute top-1 right-1 bg-white bg-opacity-80 rounded-full px-2 py-0.5 text-red-600 text-lg font-bold shadow"
            on:click={() => removeImage(idx)}
            aria-label="Remove image"
          >
            ×
          </button>
        </div>
      {/each}
    </div>
    {#if images.length === 3}
      <button class="mt-4 px-6 py-2  rounded-md" on:click={finalise}>
        Finalise
      </button>
    {/if}
</div>

{#if showCamera}
  <div class="fixed inset-0 bg-opacity-70 flex items-center justify-center z-50">
    <div class="bg-white rounded-lg  p-4 flex flex-col items-center relative">
      <video bind:this={videoEl} autoplay playsinline class="rounded w-80 h-60 bg-black"></video>
      <canvas bind:this={canvasEl} style="display:none;"></canvas>
      
      {#if availableCameras.length > 1}
        <div class="mt-2">
          <button class="plex text-sm" on:click={switchCamera}>
            Switch Camera ({availableCameras.length} available)
          </button>
        </div>
      {/if}
      
      <div class="flex gap-4 mt-4">
        <button class="plex" on:click={capturePhoto}>Capture</button>
        <button class="plex" on:click={closeCamera}>Cancel</button>
      </div>
    </div>
  </div>
{/if}
