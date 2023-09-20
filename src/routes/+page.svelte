<script>
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition'
    import { haversineDistance } from "$lib/haversine";
    import '@fontsource/dotgothic16';

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
        &page_size=1000
        &service_node=499`)
        return await response.json()
    })()

    const orderData = (results, coords) => {
        if (coords.length != 0) {
            results.forEach((result) => {
                if (result.location != null){
                    result.distance = haversineDistance([coords.latitude, coords.longitude], [result.location.coordinates[1], result.location.coordinates[0]])
                } else {
                    result.distance = 666
                }
            })

            return results.sort((a,b) => {
                return a.distance - b.distance
            })
        } else {
            return results
        }
    }
</script>

{#await getLeikkipuistot}
    <div class="loader" out:fade={{ duration: 100 }}>
        <img src="/images/kiikkuuko.gif" alt="loading image with cool swing" class="loaderImg"/>
    </div>
{:then data}
	{#each orderData(data.results, coords) as result}
    <div class="row" in:fade={{ delay: 101, duration: 100 }}>
        <p class="distance">{#if result.distance != null}~ {result.distance} km{/if}</p> 
        <p class="name">{result.name.fi}</p>
        <p class="mapLink">
            {#if result.distance != null && result.location != null}
            <a href="https://www.google.com/maps/dir/?api=1&destination={result.location.coordinates[1]},{result.location.coordinates[0]}&travelmode=walking" target="_blank">
                <img src="/images/arrow.png" alt="link to google maps" class="mapLinkImg"/>
            </a>
            {/if}
        </p>
    </div>
    {/each}
{:catch error}
	<p>An error occurred!</p>
{/await}

<style>
	.row {
        border-bottom: solid #eee 1px;
        margin: 0px;
        left: 0px;
        display: flex;
        justify-content: space-between;
        font-size:1.2rem;
        font-family: 'DotGothic16', sans-serif;
	}
    .loader {
        position: fixed;
        width: 100%;
        height: 100%;
        background-color: white;
        top: 0px;
        left: 0px;
    }

    .loaderImg {
        margin: 0;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
    }

    .distance {
        line-height: 64px;
        width: 128px;
        font-size: 1rem;
    }

    .name {
        line-height: 64px;
    }

    .mapLink {
        width: 64px;
    }

</style>