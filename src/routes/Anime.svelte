<svelte:head>
	<script src="https://cdn.fluidplayer.com/v3/current/fluidplayer.min.js"></script>
</svelte:head>

<script>
	import { onMount } from "svelte";
	export let params = {};

	let IsPlayerLoading = false;
	let vPlayerContainer = null;
	let videoPlayerElement = null;
	let fluidPlayerObj = null;
	let currentEpisode = "";

	let detail = null;

	function PlayVideo(id) {
		IsPlayerLoading = true;

		try {
			function CreatePlayer(json) {
				if (fluidPlayerObj) fluidPlayerObj.destroy();
				vPlayerContainer.innerHTML = "";

				videoPlayerElement = document.createElement("video");
				videoPlayerElement.preload = "none";
				videoPlayerElement.setAttribute("controls", "");
				vPlayerContainer.appendChild(videoPlayerElement);
				videoPlayerElement.innerHTML = `<source src="${json.sources[0].file}" type="application/x-mpegURL" />`

			    fluidPlayerObj = fluidPlayer(
			        videoPlayerElement, {
						layoutControls: {
							title: "",
							contextMenu: false,
							controlBar: { autoHideTimeout: 3, animated: true, autoHide: true },
							htmlOnPauseBlock: { html: null, height: null, width: null },
							autoPlay: false,
							mute: false,
							allowTheatre: true,
							playPauseAnimation: true,
							playbackRateEnabled: true,
							allowDownload: false,
							playButtonShowing: true,
							fillToContainer: true
						}
				});
				currentEpisode = id;
			}

			let cachedJson = localStorage.getItem(`${params.id}__${id}`);
			if (cachedJson) cachedJson = JSON.parse(cachedJson);
			if (!cachedJson || TimeDifference(new Date().getTime(), cachedJson.timestamp) >= 1) {
				console.log("No Watch Cache Found!");
				fetch(`https://gogoanime.consumet.org/vidcdn/watch/${id}`)
					.then(async function(response) {
						const json = await response.json();
						json.timestamp = new Date().getTime();
						localStorage.setItem(`${params.id}__${id}`, JSON.stringify(json));
						CreatePlayer(json);
						IsPlayerLoading = false;
					})
			} else {
				console.log("Watch Cache Found!");
				CreatePlayer(cachedJson);
				IsPlayerLoading = false;
			}
		} catch (err) {
			console.error(err);
			IsPlayerLoading = false;
		}
	}

	// Returns Difference Between 2 Dates In Hours
	function TimeDifference(date1, date2) {
		return Math.abs(date1 - date2) / 36e5; // 36e5 = 60 * 60 * 1000
	}

	onMount(function() {
		let cachedJson = localStorage.getItem(`${params.id}`);
		if (cachedJson) cachedJson = JSON.parse(cachedJson);
		if (!cachedJson || TimeDifference(new Date().getTime(), cachedJson.timestamp) >= 6) {
			console.log("No Cache Found!");
			fetch(`https://gogoanime.consumet.org/anime-details/${params.id}`)
				.then(async function(response) {
					const json = await response.json();
					json.timestamp = new Date().getTime();
					localStorage.setItem(`${params.id}`, JSON.stringify(json));
					detail = json;
					if (parseInt(detail.episodesList[0].episodeNum) > 1) {
						detail.episodesList.reverse();
					}
				})
		} else {
			console.log("Cache Found!");
			detail = cachedJson;
		}
	});
</script>

{#if detail != null}
	<div style="float: left; margin-right: 2em;">
		<img src={detail.animeImg}><br>
	</div>
	<div style="text-align: left;">
		<h1>{detail.animeTitle}</h1>
		<p>Released On: {detail.releasedDate}</p>
		<p>{detail.synopsis}</p>
	</div>
	<div style="clear: left !important;"></div>

{#if IsPlayerLoading == true}
	<p>Loading Video Player...</p>
{/if}

{#if IsPlayerLoading != true && currentEpisode && currentEpisode != ""}
	<p style="text-align: left !important;">Currently Watching: {currentEpisode.replace(/\-/g, ' ')}</p>
{/if}

	<div bind:this={vPlayerContainer}></div>
	<div class="episodeHolder">
		<h3>Episodes</h3>
		{#each detail.episodesList.reverse() as episode}
			<a class="episodeLink" on:click={ function() { PlayVideo(episode.episodeId); } }>Ep: {episode.episodeNum}</a>
		{/each}
	</div>
{:else}
	<p>Loading Anime Details...</p>
{/if}


<style>
	video {
		max-width: 80vw;
	}

	.episodeHolder {
		text-align: center;
		margin: 1em auto;
		max-width: 45em;
	}

	.episodeLink {
		user-select: none;
		display: inline-block;
		border-radius: 8px;
		border: 1px solid transparent;
		padding: 0.2em 1em;
		margin: 0.2em;
		font-size: 1em;
		font-weight: 500;
		font-family: inherit;
		background-color: #222;
		cursor: pointer;
		transition: border-color 0.25s;
	}

	.episodeLink:hover {
		outline: none;
		border-color: #646cff;
	}

	.episodeLink.currentEp {
		color: #FFF;
		outline: none;
		border: 1px solid #ffed64 !important;
	}
</style>
