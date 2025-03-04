<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import Tooltip from '../atoms/Tooltip.svelte';

	import { user } from '../../util/discord';
	import type { YouTubeMusic } from '../../util/types';

	let activity = `@${user.username}`,
		details = 'Fetching...',
		activityImage = 'default.webp',
		pulse = 30000,
		activityNumber = 0,
		state: string,
		smallImage: string,
		isSpotify: boolean,
		isYouTubeMusic: boolean,
		isActivity: boolean,
		songLink: string,
		progress: number,
		elapsed: string,
		mediaTotal: number,
		currentSetInterval: ReturnType<typeof setInterval>,
		currentRequestAnimationFrame: number,
		lanyard: WebSocket | null = null,
		worldTime: string = '',
		worldTimeInterval: ReturnType<typeof setInterval>;

	const images: { [key: string]: string } = {
		'CLIP STUDIO PAINT': 'https://i.imgur.com/IUVs3RB.png',
		'YouTube Music': 'https://i.imgur.com/mBoiikK.png'
	};

	function updateWorldTime() {
		const thTime = new Date().toLocaleString('en-US', {
			timeZone: 'Asia/Bangkok',
			hour: '2-digit',
			minute: '2-digit',
			second: '2-digit',
			hour12: true
		});
		worldTime = thTime;
	}

	function localTime() {
		state = new Date().toLocaleTimeString('en-US', { timeZone: 'Asia/Bangkok' });
	}

	function musicProgress(media: YouTubeMusic) {
		mediaTotal = media.timestamps.end - media.timestamps.start;
		progress = 100 - (100 * (media.timestamps.end - new Date().getTime())) / mediaTotal;
	}

	function elapsedTime(timestampStart: number) {
		let elapsedMs = new Date().getTime() - timestampStart;
		elapsed = new Date(elapsedMs).toISOString().slice(11, 19) + ' elapsed';
		if (elapsed.slice(0, 2) === '00') {
			elapsed = elapsed.slice(-13);
		}
	}

	function resetToDefaultState() {
		activity = `@${user.username}`;
		details = 'Fetching...';
		activityImage = 'default.webp';
		smallImage = '';
		isSpotify = false;
		isYouTubeMusic = false;
		isActivity = false;
		progress = 0;
		elapsed = '';
	}

	function connect() {
		if (currentSetInterval) clearInterval(currentSetInterval);
		if (worldTimeInterval) clearInterval(worldTimeInterval);
		if (lanyard) lanyard.close();

		resetToDefaultState();

		updateWorldTime();
		worldTimeInterval = setInterval(updateWorldTime, 1000);

		lanyard = new WebSocket('wss://api.lanyard.rest/socket');
		
		lanyard.onopen = () => {
			console.log('Synced with Discord rich presence!');
			currentSetInterval = setInterval(() => {
				if (lanyard && lanyard.readyState === WebSocket.OPEN) {
					lanyard.send(JSON.stringify({ op: 3 }));
				}
			}, 30000);
		};

		lanyard.onmessage = (e) => {
			try {
				let json = JSON.parse(e.data);
				let opcode = json.op;
				let data = json.d;

				if (opcode === 1) {
					pulse = data.heartbeat_interval;
					lanyard.send(
						JSON.stringify({
							op: 2,
							d: { subscribe_to_id: user.id }
						})
					);
				}

				function tick() {
					try {
						if (isSpotify) musicProgress(data.spotify);
						else if (isYouTubeMusic) musicProgress(data.youtube_music);
						else if (isActivity) elapsedTime(data.activities[activityNumber].timestamps.start);
						else localTime();
						currentRequestAnimationFrame = requestAnimationFrame(tick);
					} catch (tickError) {
						console.error('Tick error:', tickError);
						cancelAnimationFrame(currentRequestAnimationFrame);
					}
				}

				if (opcode === 0) {
					isSpotify = data.listening_to_spotify && data.spotify;
					isYouTubeMusic = data.activities && data.activities.some((act: { name: string }) => act.name === 'YouTube Music');
					isActivity = data.activities && data.activities.length > 0;

					if (isSpotify) {
						({ 
							song: activity, 
							artist: details, 
							album: state, 
							album_art_url: activityImage 
						} = data.spotify);

						details = 'by ' + details.replace(/;/g, ',');
						state = activity === state ? '' : 'on ' + state;
						songLink = `https://open.spotify.com/track/${data.spotify.track_id}`;
						smallImage = '';
						cancelAnimationFrame(currentRequestAnimationFrame);
						tick();
					} else if (isYouTubeMusic) {
						const ytmIndex = data.activities.findIndex((act: { name: string }) => act.name === 'YouTube Music');
						if (ytmIndex !== -1) {
							const ytmActivity = data.activities[ytmIndex];
							activity = ytmActivity.details || 'Unknown Song';
							details = ytmActivity.state ? 'by ' + ytmActivity.state.replace(/;/g, ',') : '';
							
							activityImage = ytmActivity.assets && ytmActivity.assets.large_image
								? `https://cdn.discordapp.com/app-assets/${ytmActivity.application_id}/${ytmActivity.assets.large_image}.webp?size=512`
								: images['YouTube Music'];
							
							songLink = ytmActivity.sync_id 
								? `https://music.youtube.com/watch?v=${ytmActivity.sync_id}` 
								: '';
							
							smallImage = ytmActivity.assets && ytmActivity.assets.small_image
								? `https://cdn.discordapp.com/app-assets/${ytmActivity.application_id}/${ytmActivity.assets.small_image}.webp?size=512`
								: '';
							
							if (ytmActivity.timestamps) {
								data.youtube_music = {
									track_id: ytmActivity.sync_id || '',
									timestamps: ytmActivity.timestamps,
									song: activity,
									artist: details.replace('by ', ''),
									album_art_url: activityImage
								};
							}
							
							cancelAnimationFrame(currentRequestAnimationFrame);
							tick();
						}
					} else if (isActivity) {
						const currentActivity = data.activities[activityNumber];
						activity = currentActivity.name;
						details = currentActivity.details || '';
						state = currentActivity.state || '';
						
						activityImage = currentActivity.assets
							? `https://cdn.discordapp.com/app-assets/${currentActivity.application_id}/${currentActivity.assets.large_image}.webp?size=512`
							: images[activity] || 'question_mark.png';
						
						smallImage = currentActivity.assets && currentActivity.assets.small_image
							? `https://cdn.discordapp.com/app-assets/${currentActivity.application_id}/${currentActivity.assets.small_image}.webp?size=512`
							: '';
						
						cancelAnimationFrame(currentRequestAnimationFrame);
						tick();
					} else {
						activity = `@${user.username}`;
						details = data.discord_status.charAt(0).toUpperCase() + data.discord_status.slice(1);
						details = details === 'Dnd' ? 'Do Not Disturb' : details;
						activityImage = 'default.webp';
						smallImage = '';
						cancelAnimationFrame(currentRequestAnimationFrame);
						tick();
					}
				}
			} catch (parseError) {
				console.error('Parsing error:', parseError);
				resetToDefaultState();
			}
		};

		lanyard.onclose = (event) => {
			console.error('WebSocket closed:', event);
			if (currentSetInterval) clearInterval(currentSetInterval);
			setTimeout(connect, 5000);
		};

		lanyard.onerror = (error) => {
			console.error('WebSocket error:', error);
			resetToDefaultState();
		};
	}

	onMount(connect);

	onDestroy(() => {
		if (currentSetInterval) clearInterval(currentSetInterval);
		if (worldTimeInterval) clearInterval(worldTimeInterval);
		if (lanyard) lanyard.close();
		if (currentRequestAnimationFrame) cancelAnimationFrame(currentRequestAnimationFrame);
	});
</script>

<h2>activity</h2>
<div class="contain">
	<img src={activityImage} alt={activity} class="big" class:spin={isSpotify || isYouTubeMusic} />
	{#if smallImage}
		<img src={smallImage} alt={activity} class="small" />
	{/if}
	<div>
		{#if isSpotify || (isYouTubeMusic && songLink)}
			<a href={songLink} target="_blank" rel="noreferrer">
				<Tooltip tip={isSpotify ? "Open Spotify" : "Open YouTube Music"}>
					<h3>{activity}</h3>
				</Tooltip>
			</a>
		{:else}
			<h3>{activity}</h3>
		{/if}
		<h5>{details || ''}</h5>
		<h5>{state || ''}</h5>
		{#if isSpotify || isYouTubeMusic}
			<progress max="100" value={progress} />
		{:else if isActivity}
			<h5>{elapsed}</h5>
		{/if}
		
		<!-- New World Time Display -->
		<h5 class="world-time">🇹🇭 {worldTime}</h5>
	</div>
</div>

<style lang="scss">
	.contain {
		display: flex;
		gap: 2.25rem;
		align-items: center;
	}

	a,
	a:not(:hover) {
		border-radius: 4px;
		text-decoration: underline;
		text-decoration-color: var(--bg-color);
		transition: 0.3s var(--bezier-one);
	}

	h2 {
		display: none;
	}

	a:hover {
		text-decoration-color: var(--text-primary);
	}

	.big {
		height: 135px;
		width: 135px;
		border-radius: 20px;
		display: relative;
		user-select: none;
		transition: all 0.3s var(--bezier-one);
	}

	.small {
		height: 40px;
		width: 40px;
		border-radius: 50%;
		position: absolute;
		transform: translate(275%, 150%);
		outline: 6px solid var(--bg-color);
		background-color: var(--bg-color);
	}

	progress {
		-webkit-appearance: none;
		-moz-appearance: none;
		appearance: none;
		border: 0;
		border-radius: 10rem;
		margin: 0;
		margin-top: 0.6rem;
		background-color: var(--elevation-one);
		height: 0.6rem;
		overflow: hidden;

		&::-webkit-progress-bar {
			background-color: var(--elevation-one);
			border-radius: 10rem;
		}

		&[value]::-webkit-progress-value {
			background-color: var(--accent);
			border-radius: 10rem;
		}

		&[value]::-moz-progress-bar {
			background-color: var(--accent);
			border-radius: 10rem;
		}
	}

	@keyframes rotate {
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	}

	.spin {
		animation: rotate 40s linear infinite;
		border-radius: 50%;
	}

	.world-time {
		margin-top: 0.5rem;
		color: var(--text-secondary);
		font-size: 0.9rem;
	}

	@media screen and (max-width: 868px) {
		h2 {
			display: block;
			margin-bottom: 1rem;
		}
		div {
			justify-content: left;
		}

		.big {
			height: 100px;
			width: 100px;
			border-radius: 17px;
		}

		.spin {
			border-radius: 50%;
		}

		.small {
			transform: translate(190%, 110%);
		}
	}
</style>