<script>
	// autocomplete lib to make ux more appealing
	import AutoComplete from "simple-svelte-autocomplete";
	// Just a formatting object 
	import Song from "./Song.svelte";

	// token to authenticate spotify api 
	let access_token = 'BQAGMYCQo2tCW6HdOEKDBN_BFu33sfmeRcvQDAUFKiE7KWAVOFqXDxAXDfSOQGSZ5fMs70g75YprVQhqp4k';

	let genBtnDisabled = true; // svelte allows embedding boolean variables so we can create a global control
	let currentSong = {}; // this let's us store the current song object so we can access it in the next function
	// Svelte will automatically run this code every time currentSong changes 
	$: {
		genBtnDisabled = currentSong.name == undefined;
	}

	var results = []; // we store the api results here
	// ^ put results below the $: because we don't want to run the code every time the results change

	const API_URL = "https://api.spotify.com/v1/"; // might change if spotify api bumps major version

	// this includes my list filtering because spotify sometimes returns things that aren't tracks
	const fmt_tracks = (tracks) => {
		var tracks_list = [];
		for (var track of tracks) { // using forEach loop
			if (track.name == undefined) {
				continue; // filtering the list to ignore tracks without a name field to avoid errors
				// These are playlists, albums, etc.
			} else {
				let pretty = `${track.name} - ${track.artists
					.map((artist) => artist.name)
					.join(", ")}`;
				if (pretty.length >= 35) {
					pretty = pretty.substring(0, 35) + "...";
				}
				tracks_list.push( {
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
				});
			}
		}	
		console.log(tracks_list);
		return tracks_list;	
	}

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
			return fmt_tracks(data.tracks.items);
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
			results = fmt_tracks(data.tracks);
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
				localFiltering=false
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

	#gen-btn:hover:enabled {
		transform: scale(1.01);
	}
</style>
