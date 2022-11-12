<script>
	let IsLoading = false;
	let searchQuery = "";
	let searchResults = [];

	async function _SearchQuery(e) {
		if (searchQuery) {
			IsLoading = true;
			// Replace Multiple Space in the query with "-"
			const url = `https://gogoanime.consumet.org/search?keyw=${searchQuery.replace(/\s+/g, "-")}` 
			const response = await fetch(url, { cache: "reload" });
			const json = await response.json();
			searchResults = json;
			IsLoading = false;
		}
	}
</script>

<h1>AnimeGoGo</h1>
<input bind:value={searchQuery} on:keypress={function(e) { if (e.code == "Enter") { _SearchQuery(e) } }} placeholder="Search For Anime..." type="text" name="search-box">
<button on:click={_SearchQuery}>Search 🔍</button>

{#if IsLoading == true}
	<p>Loading...</p>
{/if}

{#if searchResults.length > 0}
	<div>
	{#each searchResults as item}
		<a href={`${location.pathname}#/anime/${item.animeId}`} class="searchItem">
			<img class="searchItemImage" src={item.animeImg}>
			<p class="searchItemTitle">{item.animeTitle}</p>
		</a>
	{/each}
	</div>
{/if}

<style>
	.searchItem {
		cursor: pointer;
		display: inline-block;
		padding: 0.8em 1em;
		margin: 1em;
		text-align: center;
		background: #333;
		border-radius: 0.5em;
	}

	.searchItemImage {
		margin: 0 auto;
		display: block;
		width: 250px;
		max-width: 250px;
		padding: 0.5em 0;
	}

	.searchItemTitle {
		margin: 0.5em 1em;
		color: rgba(255, 255, 255, 0.87);
		display: block;
	}
</style>
