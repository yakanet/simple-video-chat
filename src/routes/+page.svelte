<script lang="ts">
	import type { Peer } from 'peerjs';
	import { onMount, tick } from 'svelte';
	let peer: Peer;
	let client_code: string;
	let destination_code: string;
	let clientVideo: HTMLVideoElement;
	let destinationVideo: HTMLVideoElement;
	let calling = false;

	async function createPeer() {
		const { Peer } = await import('peerjs');
		peer = new Peer();
		peer.on('open', (id) => {
			client_code = id;
		});
		peer.on('call', async (call) => {
			// Answer the call, providing our mediaStream
			const videoStream = await getMediaStream();
			call.answer(videoStream);
			calling = true;
            await tick();
            clientVideo.srcObject = videoStream;

			call.on('stream', (stream) => {
				// `stream` is the MediaStream of the remote peer.
				// Here you'd add it to an HTML video/canvas element.
				destinationVideo.srcObject = stream;
			});
		});
	}

	async function call(code: string) {
		const videoStream = await getMediaStream();

		const call = peer.call(code, videoStream);
		calling = true;
        await tick();
        clientVideo.srcObject = videoStream;

		call.on('close', () => {
			calling = false;
		});
		call.on('stream', (stream) => {
			// `stream` is the MediaStream of the remote peer.
			// Here you'd add it to an HTML video/canvas element.
			destinationVideo.srcObject = stream;
		});
	}

	function getMediaStream(): Promise<MediaStream> {
		return navigator.mediaDevices.getUserMedia({
			audio: true,
			video: {
				facingMode: 'user'
			}
		});
	}

	onMount(async () => {
		createPeer();
	});
</script>

{#if client_code}
	<p>Mon code : {client_code}</p>
	<input type="text" bind:value={destination_code} />
	<button type="button" on:click={() => call(destination_code)}>Call</button>
{/if}
{#if calling}
	<video class="me" muted autoplay bind:this={clientVideo} />
	<video class="other" autoplay bind:this={destinationVideo} />
{/if}

<style lang="scss">
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: content-box;
	}
	:global(html) {
		font-family:
			system-ui,
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			Roboto,
			Oxygen,
			Ubuntu,
			Cantarell,
			'Open Sans',
			'Helvetica Neue',
			sans-serif;
	}

	video.other {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		background: black;
	}

    video.me {
        position: absolute;
        bottom: 1rem;
        right: 1rem;
        max-width: min(30%, 300px);
        background: black;
        z-index: 1;
    }
</style>
