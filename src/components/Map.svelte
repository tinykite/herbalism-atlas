<script lang="ts">
	import { onMount } from 'svelte';
	import maplibregl from 'maplibre-gl';
	import * as pmtiles from 'pmtiles';
	import { layersWithCustomTheme } from 'protomaps-themes-base';
	import { HERBALISM_THEME } from '$lib/herbalismTheme';
	import { PUBLIC_MAP_TILE_URL } from '$env/static/public';

	const { Map, Marker, Popup, LngLat } = maplibregl;

	interface Props {
		points: Array<any> | undefined;
		center: Array<Number>;
		zoom: Number;
	}

	/* DEBUG ONLY */
	// let infoDiv: HTMLElement;
	let { points, center, zoom } = $props();
	let mapContainer: HTMLElement | null | undefined = $state();
	let map = $state();

	const protocol = new pmtiles.Protocol();
	maplibregl.addProtocol('pmtiles', protocol.tile);

	interface mapPoint {
		Latitude: string;
		Longitude: string;
		Name: string;
		CityState: string;
	}

	$effect(() => {
		if (!!map) {
			const { lat, lng } = map.getCenter();

			if (lng !== parseFloat(center[0]) || lat !== parseFloat(center[1])) {
				map.flyTo({
					zoom,
					center
				});
			}
		}
	});

	let popupOffsets = {
		bottom: [0, -30]
	};

	onMount(() => {
		map = new Map({
			container: mapContainer as HTMLElement,
			style: {
				version: 8,
				glyphs: 'https://protomaps.github.io/basemaps-assets/fonts/{fontstack}/{range}.pbf',
				sources: {
					protomaps: {
						type: 'vector',
						tiles: [PUBLIC_MAP_TILE_URL],
						maxzoom: 6,
						attribution:
							'<a href="https://protomaps.com">Protomaps</a> © <a href="https://openstreetmap.org">OpenStreetMap</a>'
					}
				},
				layers: layersWithCustomTheme('protomaps', HERBALISM_THEME, 'en')
			},
			center,
			zoom,
			maxZoom: 10
		});

		/* DEBUG ONLY */
		// map.on('move', updateInfo);
		// map.on('zoom', updateInfo);
		// updateInfo();

		points.map((point: mapPoint) => {
			const coords = new LngLat(parseFloat(point.Longitude), parseFloat(point.Latitude));
			const popup = new Popup({ offset: popupOffsets }).setHTML(
				`<div><h3>${point.Name}</h3><p>${point.CityState}</p></div>`
			);
			new Marker({ color: '#743A78', scale: 0.85 }).setLngLat(coords).setPopup(popup).addTo(map);
		});

		map.on('load', async () => {
			mapContainer?.style.setProperty('opacity', 1);
		});
	});

	/* DEBUG ONLY */
	// const updateInfo = () => {
	// 	const center = map.getCenter();
	// 	const zoom = map.getZoom();
	// 	infoDiv.textContent = `Lat: ${center.lat.toFixed(4)}, Lng: ${center.lng.toFixed(
	// 		4
	// 	)}, Zoom: ${zoom.toFixed(2)}`;
	// };
</script>

<div class="map">
	<div class="map__graphic" bind:this={mapContainer}></div>
</div>

<!-- 
<div class="mapInfo" bind:this={infoDiv} /> -->

<style>
	.map {
		position: fixed;
		height: 100%;
		width: 100%;
	}
	.map__graphic {
		height: 100vh;
		opacity: 0;
		transition: 0.3s ease;
	}

	.mapInfo {
		position: fixed;
		top: 20px;
		z-index: 700;
		background: white;
	}

	:global(.maplibregl-popup-content) {
		display: grid;
		grid-template-columns: 1fr 0.5rem;
		align-items: center;
		padding-block-start: 1.25rem;
	}
</style>
