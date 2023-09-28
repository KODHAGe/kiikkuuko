<script>
    import { onMount } from 'svelte';
    import { fade } from 'svelte/transition'
    import { haversineDistance } from "$lib/haversine";
    import '@fontsource/dotgothic16';

    let coords = [],
        places = {}

    onMount(async () => {
        let localData = localStorage.getItem("places");
        if (!localData) {
            const response = await fetch(`https://api.hel.fi/servicemap/v2/unit/?
            format=json
            &geometry=true
            &include=service_nodes%2Cservices%2Caccessibility_properties%2Cdepartment%2Croot_department&language=fi
            &only=street_address%2Clocation%2Cname%2Cmunicipality%2Caccessibility_shortcoming_count%2Cservice_nodes%2Ccontract_type%2Corganizer_type
            &page=1
            &page_size=1000
            &service_node=499`)
            places = await response.json()
            localStorage.setItem("places", JSON.stringify(places))
        } else {
            places = JSON.parse(localData)
        }

        if ("geolocation" in navigator) {
            navigator.geolocation.getCurrentPosition((position) => {
                coords = position.coords
            });
        } else {
            /* geolocation IS NOT available */
        }
    })

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
{#if places.results}
    {#each orderData(places.results, coords) || [] as result }
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
{:else}
    <div class="loader" out:fade={{ duration: 100 }}>
        <img src="/images/kiikkuuko.gif" alt="loading image with cool swing" class="loaderImg"/>
    </div>
{/if}
<style>
	.row {
        border-bottom: solid #eee 1px;
        margin: 0px;
        left: 0px;
        display: flex;
        justify-content: space-between;
        font-size:1.2em;
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
        width: 90px;
        font-size: 0.6em;
        flex: none;
        display: flex;
        align-content: flex-end;
        flex-direction: column;
        flex-wrap: nowrap;
        align-items: center;
        justify-content: center;
    }

    .name {
        width:50%;
        text-align: left;
        display: flex;
        align-content: center;
        flex-wrap: wrap;
    }

    .mapLink {
        width: 64px;
        display: flex;
        align-content: center;
        flex-wrap: wrap;
    }

    .mapLink a {
        display: flex;
        align-content: center;
        flex-wrap: wrap;
        flex-direction: column;
    }

</style>