<script>
    import { fade, fly, slide } from "svelte/transition";

    export let name;
    export let artists;
    export let id;
    export let delay;
    let visible = false;
    setTimeout(() => {
        visible = true;
    }, delay);

    let winHeight;
</script>

<svelte:window bind:innerHeight={winHeight} />
{#if visible}
    <div {id} class="song" transition:fly|local={{ y: winHeight, duration: 1500 }} >
        <div class="container" id="song">
            <div class="columns">
                <div class="column">
                    <div class="card">
                        <div class="card-content">
                            <div class="media">
                                <div class="media-left">
                                    <figure class="image is-96x96">
                                        <!-- <img src={album} alt="Placeholder" /> -->
                                        <iframe
                                            title={name}
                                            src={`https://open.spotify.com/embed/track/${id}`}
                                            width="85%"
                                            height="85%"
                                            frameborder="0"
                                            allowtransparency="true"
                                            allow="encrypted-media"
                                        />
                                    </figure>
                                </div>
                                <div class="media-content">
                                    <p class="title is-2 has-text-weight-bold">
                                        {name}
                                    </p>
                                    <p class="subtitle is-6">
                                        by <strong>{artists}</strong>
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
{/if}

<style>
    .song {
        transition: all 0.2s ease-in-out;
    }

    .song:hover {
        transform: scale(1.01);
    }
</style>
