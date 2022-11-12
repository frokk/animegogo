<script>
	import Router from "svelte-spa-router";

	import Home from "./routes/Home.svelte";
	import Anime from "./routes/Anime.svelte";
	import NotFound from "./routes/NotFound.svelte";
	import Footer from "./lib/Footer.svelte";

	let routes = {
		"/": Home,
		"/anime/:id": Anime, // Takes Anime Movie/Series ID
		"*": NotFound
	}
</script>

<main>
	<Router {routes} />
</main>
<Footer />

<style>
	main {
		min-height: 80vh;
	}
</style>

