<script>
    import { onDestroy, onMount } from 'svelte';


    let latitude, longitude, errorMsg;
    let previousLatitude, previousLongitude;
    let isMoving = false;
    let watchId;
    let map;
    let marker;
    let mapInitialized = false;

    async function loadLocation() {
         if (typeof window !== 'undefined' && navigator && navigator.geolocation){
            const L = await import('leaflet');
            const success = (position) => {
                previousLatitude = latitude;
                previousLongitude = longitude;

                latitude = position.coords.latitude;
                longitude = position.coords.longitude;

                isMoving = (latitude !== previousLatitude) || (longitude !== previousLongitude);

                if (!mapInitialized) {
                    map = L.map('map').setView([latitude, longitude], 13);

                    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
                    }).addTo(map);

                    mapInitialized = true;
                } else {
                    map.setView([latitude, longitude], 13);
                }

                if (marker) {
                    marker.setLatLng([latitude, longitude]);
                } else {
                    marker = L.marker([latitude, longitude]).addTo(map);
                }
            };

            const error = (err) => {
                errorMsg = `ERROR(${err.code}): ${err.message}`;
            };

            const options = {
                enableHighAccuracy: true,
                maximumAge: 30000,
                timeout: 27000
            };

            watchId = navigator.geolocation.watchPosition(success, error, options);
        }
    }

    onMount(loadLocation);

    onDestroy(() => {
        if (watchId) {
            navigator.geolocation.clearWatch(watchId);
        }
        if (map) {
            map.remove();
        }
    });
</script>

<div>
    {#if errorMsg}
        <p>{errorMsg}</p>
    {:else}
        <p>Latitude: {latitude}</p>
        <p>Longitude: {longitude}</p>
        <p>Previous Latitude: {previousLatitude}</p>
        <p>Previous Longitude: {previousLongitude}</p>
        <p>Moving: {isMoving ? 'Yes' : 'No'}</p>
    {/if}
    <div id="map" style="height: 60vh;"></div>
</div>

<button on:click={loadLocation}>Reload</button>
