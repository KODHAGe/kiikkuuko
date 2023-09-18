<script>
    //import Geolocation from "svelte-geolocation"
    import { onMount } from 'svelte';
    import { haversineDistance } from "$lib/haversine";

    let coords = []
    onMount(async () => {
        if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition((position) => {
            coords = position.coords
        });
        } else {
        /* geolocation IS NOT available */
        }
    })
    
    const getLeikkipuistot = (async () => {
        const response = await fetch(`https://api.hel.fi/servicemap/v2/unit/?
        format=json
        &geometry=true
        &include=service_nodes%2Cservices%2Caccessibility_properties%2Cdepartment%2Croot_department&language=fi
        &only=street_address%2Clocation%2Cname%2Cmunicipality%2Caccessibility_shortcoming_count%2Cservice_nodes%2Ccontract_type%2Corganizer_type
        &page=1
        &page_size=10
        &service_node=499`)
        return await response.json()
    })()
</script>

<pre>{JSON.stringify(coords.latitude + " " + coords.longitude)}</pre>
{#await getLeikkipuistot}
	<p>...waiting</p>
{:then data}
	{#each data.results as result}
    <div class="row">
        <p>{result.name.fi}</p>
        {#if result.location != null}
            <div>{result.location.coordinates[1]}</div>
            <a href="https://www.google.com/maps/search/?api=1&query={result.location.coordinates[1]},{result.location.coordinates[0]}" target="_blank">link</a>
            <div>{haversineDistance([coords.latitude, coords.longitude], [result.location.coordinates[1], result.location.coordinates[0]])} metriä</div>
        {/if}
    </div>
    {/each}
{:catch error}
	<p>An error occurred!</p>
{/await}

<style>
	.row {
		background-color: #eee;
        border-bottom: solid 1px;
        margin: 0px;
	}
</style>