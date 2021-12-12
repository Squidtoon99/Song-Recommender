<script>
	import AutoComplete from "simple-svelte-autocomplete";
	import Song from "./Song.svelte";

	let access_token =
		"BQD7E9RF_uelZRCITjrl3aaUIGtCcbrT7j6H7BRyRYrZEBVYQvbixq68wx6pq3M_8JwlrgPgt_EMw36b3BI";

	let genBtnDisabled = true;
	let currentSong = {};
	$: {
		genBtnDisabled = currentSong.name == undefined;
	}

	var results = [];

	const API_URL = "https://api.spotify.com/v1/";
	const findSongs = async (query) => {
		var url = new URL(API_URL + "search?");
		var params = new URLSearchParams({
			q: query,
			type: "track",
			limit: 5,
		});

		const res = await fetch(url + params, {
			headers: {
				Authorization: "Bearer " + access_token,
			},
		});
		console.log(res);
		if (res.ok) {
			const data = await res.json();
			console.log(data);
			return data.tracks.items.map((track) => {
				let pretty = `${track.name} - ${track.artists
					.map((artist) => artist.name)
					.join(", ")}`;
				if (pretty.length >= 35) {
					pretty = pretty.substring(0, 35) + "...";
				}
				return {
					id: track.id,
					name: track.name,
					artists: track.artists
						.map((artist) => artist.name)
						.join(", "),
					album: track.album.name,
					image: track.album.images[0].url,
					preview: track.preview_url,
					uri: track.uri,
					pretty: pretty,
				};
			});
		} else {
			return [];
		}
	};

	const generateSongs = async () => {
		results = [];

		const url = new URL(API_URL + "recommendations?");
		var params = new URLSearchParams({
			seed_tracks: currentSong.id,
			limit: 8,
		});

		const res = await fetch(url + params, {
			headers: {
				Authorization: "Bearer " + access_token,
			},
		});
		console.log(res);
		currentSong = {};
		genBtnDisabled = false;
		document.getElementById("gen-btn").classList.remove("is-loading");
		if (res.ok) {
			const data = await res.json();
			console.log(data);
			results = data.tracks.map((track) => {
				let pretty = `${track.name} - ${track.artists
					.map((artist) => artist.name)
					.join(", ")}`;
				if (pretty.length >= 35) {
					pretty = pretty.substring(0, 35) + "...";
				}
				return {
					id: track.id,
					name: track.name,
					artists: track.artists
						.map((artist) => artist.name)
						.join(", "),
					album: track.album.name,
					image: track.album.images[0].url,
					preview: track.preview_url,
					uri: track.uri,
					pretty: pretty,
				};
			});
		}
	};
</script>

<section class="section">
	<div class="container">
		<div class="columns is-centered">
			<h1 class="title is-2 has-text-white">Song Recommender</h1>
		</div>
	</div>

	<div class="container is-fluid">
		<div id="song-input" class="section">
			<AutoComplete
				placeholder="Search for a song"
				class="input"
				searchFunction={findSongs}
				delay="500"
				localFiltering="false"
				labelFieldName="pretty"
				valueFieldName="id"
				bind:selectedItem={currentSong}
			/>

			<button
				id="gen-btn"
				on:click={(e) => {
					e.target.classList.add("is-loading");
					genBtnDisabled = true;

					return generateSongs();
				}}
				class="button mt-2 w-auto is-fullwidth is-primary"
				disabled={genBtnDisabled}
				style={`background: -webkit-linear-gradient(45deg, #8a86db, ${
					genBtnDisabled ? "red" : "#00ff95"
				} 80%);`}
			>
				<span class="icon is-small">
					<i class="fas fa-search" />
				</span>
				<span> Generate </span>
			</button>
		</div>
	</div>
	{#if results.length > 0}
		{#each results as song, i}
			<div class="container">
				<Song
					id={song.id}
					name={song.name}
					artists={song.artists}
					album={song.image}
					delay={i * 300}
				/>
			</div>
		{/each}
	{/if}
</section>

<style>
	:global(body) {
		background: #181818;
		color: #ffffff;
	}
	:global(section) {
		background: #181818;
		color: #ffffff;
	}

	#gen-btn {
		background: -webkit-linear-gradient(45deg, #8a86db, red 80%);
		border: 0;
		color: white;
	}
</style>
