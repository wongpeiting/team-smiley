<script>
	import PTChart from './stories/PTChart.svelte';

	// Copy _StoryCardTemplate.svelte into stories/, rename it, then import here:
	// import YourNameStory from './stories/YourNameStory.svelte';
</script>

<main class="blog">
	<div class="blog-column">
		<header class="blog-header">
			<h1 class="blog-title">Creepy Roommates</h1>
			<p class="blog-dek">Rats. Roaches. Bed bugs. Mosquitoes. What city data reveals about the creatures that share New York with 8 million people.</p>
			<p class="blog-date">Published June 17, 2026</p>
		</header>

		<PTChart />
		<!-- <YourNameStory /> -->
	</div>
</main>

<style>
	.blog {
		min-height: 100vh;
		padding: 3rem 1.5rem 4rem;
		background: #fff;
		color: #111;
		font-family: 'Source Sans 3', sans-serif;
	}

	.blog-column {
		max-width: 42rem;
		margin: 0 auto;
	}

	.blog-header {
		margin-bottom: 3rem;
	}

	.blog-title {
		font-family: 'Source Serif 4', serif;
		font-size: 2.75rem;
		font-weight: 900;
		line-height: 1.1;
		margin: 0 0 0.75rem;
	}

	.blog-dek {
		font-size: 1.125rem;
		line-height: 1.6;
		color: #444;
		margin: 0 0 0.75rem;
	}

	.blog-date {
		font-size: 0.875rem;
		color: #666;
		margin: 0;
	}
</style>
