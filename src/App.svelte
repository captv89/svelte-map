<script>
	import L from 'leaflet';
	import MapToolbar from './MapToolbar.svelte';
	import MarkerPopup from './MarkerPopup.svelte';
	import * as markerIcons from './markers.js';
	let map;

	const markerLocations = [
		[54.172658524532714, 11.998293346275874],
		[54.07890412395562, 12.064211314467363],
		[54.05694382599608, 12.116739695369954],
		[54.0732640549763, 12.189867441332385],
		[54.117558242775566, 12.170641367276534],
		[54.156377388742904, 12.136309092176802],
		[54.183308146335655, 12.124292795891893],
	];

	const myFavLocations = [
		{
			place: 'Rostock',
			description:
				'Rostock is a city in Mecklenburg-Vorpommern, Germany, and the largest city in the state. It is situated on the Warnow river north of Berlin and is the capital of the Land of Mecklenburg-Vorpommern. The city is home to the University of Rostock, the oldest university in Northern Europe. Rostock is a major seaport and an important intermodal transport hub. The city is also a tourism destination, with its historic old town and many beaches on the Baltic Sea.',
			position: [54.172658524532714, 11.998293346275874],
		},
		{
			place: 'Kiel',
			description:
				"Kiel is a city in the northern German state of Schleswig-Holstein, on the Baltic Sea coast. It's known for its canals, beaches and marinas. The Holsten Gate, a 13th-century brick Gothic gate, is a landmark of the city center. The Kiel Canal, a waterway connecting the North Sea and Baltic Sea, runs through the city. The Maritime Museum has a collection of historic ships.",
			position: [54.3233, 10.1227],
		},
		{
			place: 'Hamburg',
			description:
				"Hamburg is a German port city and the second-largest city in Germany after Berlin. It's known for its canals, riverside promenades and maritime history. The Speicherstadt warehouse district is a UNESCO World Heritage Site. The Elbphilharmonie concert hall, with its distinctive glass façade, is a major landmark. The city's Reeperbahn entertainment district is home to nightclubs, bars and theaters.",
			position: [53.5511, 9.9937],
		},
		{
			place: 'Berlin',
			description:
				"Berlin is the capital and largest city of Germany by both area and population. Its 3.7 million inhabitants make it the second most populous city proper of the European Union after London. The city is one of Germany's 16 federal states. It is surrounded by the state of Brandenburg, and contiguous with Potsdam, Brandenburg's capital. The two cities are at the center of the Berlin-Brandenburg capital region, which is, with about six million inhabitants and an area of more than 30,000 km2, Germany's third-largest metropolitan region after the Rhine-Ruhr and Rhine-Main regions.",
			position: [52.52, 13.405],
		},
		{
			place: 'Munich',
			description:
				"Munich is the capital and most populous city of the German state of Bavaria, on the banks of the River Isar north of the Bavarian Alps. The city's name derives from the Old/Middle High German term Munichen, meaning 'by the monks', referring to the Benedictine monks of the former Abbey of Our Lady in Munich, who founded the city in 1158.",
			position: [48.1372, 11.5755],
		},
		{
			place: 'Frankfurt',
			description:
				"Frankfurt is a metropolis and the largest city in the German state of Hesse and the fifth-largest city in Germany. It is the financial and transportation centre of Germany and the largest financial centre in continental Europe. Frankfurt Airport is among the world's busiest. The city is home to the European Central Bank, Deutsche Bundesbank, Frankfurt Stock Exchange and several large commercial banks.",
			position: [50.1109, 8.6821],
		},
		{
			place: 'Cologne',
			description:
				"Cologne is a city in western Germany. It's known for its Gothic cathedral, the 12th-century Romanesque Church of Our Lady and the baroque Hohenzollern Bridge. The city's Old Town is home to medieval buildings, cafés and shops. The Cologne Chocolate Museum explores the city's history as a center of chocolate production. The city's Rhine River waterfront is lined with parks and promenades.",
			position: [50.9384, 6.9598],
		},
		{
			place: 'Düsseldorf',
			description:
				"Düsseldorf is a city in western Germany, capital of the state of North Rhine-Westphalia. It's known for its fashion and trade fairs, such as the Düsseldorf Art Fair and the Düsseldorf Caravan Salon. The city's Altstadt (Old Town) is home to the Gothic-style Königsallee, lined with designer shops and cafés. The Rhine River flows through the city, and the Altstadt is connected to the modern district by the Konigsufer Bridge.",
			position: [51.2277, 6.7735],
		},
	];

	const initialView = [52.52, 13.405];
	function createMap(container) {
		let m = L.map(container, { preferCanvas: true }).setView(
			initialView,
			6
		);
		L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
			attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OpenStreetMap</a>`,
			subdomains: 'abcd',
			maxZoom: 18,
		}).addTo(m);

		return m;
	}

	let eye = true;
	// let lines = true;

	let toolbar = L.control({ position: 'topright' });
	let toolbarComponent;
	toolbar.onAdd = (map) => {
		let div = L.DomUtil.create('div');
		toolbarComponent = new MapToolbar({
			target: div,
			props: {},
		});

		toolbarComponent.$on('click-eye', ({ detail }) => (eye = detail));
		// toolbarComponent.$on('click-lines', ({ detail }) => lines = detail);
		toolbarComponent.$on('click-reset', () => {
			map.setView(initialView, 5, { animate: true });
		});

		return div;
	};

	toolbar.onRemove = () => {
		if (toolbarComponent) {
			toolbarComponent.$destroy();
			toolbarComponent = null;
		}
	};

	// Create a popup with a Svelte component inside it and handle removal when the popup is torn down.
	// `createFn` will be called whenever the popup is being created, and should create and return the component.
	function bindPopup(marker, createFn) {
		let popupComponent;
		marker.bindPopup(() => {
			let container = L.DomUtil.create('div');
			popupComponent = createFn(container);
			return container;
		});

		marker.on('popupclose', () => {
			if (popupComponent) {
				let old = popupComponent;
				popupComponent = null;
				// Wait to destroy until after the fadeout completes.
				setTimeout(() => {
					old.$destroy();
				}, 500);
			}
		});
	}

	let markers = new Map();

	function markerIcon(place) {
		let html = `<div class="map-marker"><div>${markerIcons.location}</div><div class="marker-text">${place}</div></div>`;
		return L.divIcon({
			html,
			className: 'map-marker',
		});
	}

	function createMarker(place, desc, loc) {
		let icon = markerIcon(place);
		let marker = L.marker(loc, { icon });
		bindPopup(marker, (m) => {
			let c = new MarkerPopup({
				target: m,
				props: {
					place,
					desc,
				},
			});

			// c.$on('change', ({detail}) => {
			// 	count = detail;
			// 	marker.setIcon(markerIcon(count));
			// });

			return c;
		});

		return marker;
	}

	// function createLines() {
	// 	return L.polyline(markerLocations, { color: '#E4E', opacity: 0.5 });
	// }

	let markerLayers;
	// let lineLayers;
	function mapAction(container) {
		map = createMap(container);
		toolbar.addTo(map);

		markerLayers = L.layerGroup();
		for (let location of myFavLocations) {
			console.log(location.place);
			let m = createMarker(
				location.place,
				location.description,
				location.position
			);
			markerLayers.addLayer(m);
		}

		// lineLayers = createLines();

		markerLayers.addTo(map);
		// lineLayers.addTo(map);

		return {
			destroy: () => {
				toolbar.remove();
				map.remove();
				map = null;
			},
		};
	}

	// We could do these in the toolbar's click handler but this is an example
	// of modifying the map with reactive syntax.
	$: if (markerLayers && map) {
		if (eye) {
			markerLayers.addTo(map);
		} else {
			markerLayers.remove();
		}
	}

	// $: if(lineLayers && map) {
	// 	if(lines) {
	// 		lineLayers.addTo(map);
	// 	} else {
	// 		lineLayers.remove();
	// 	}
	// }

	function resizeMap() {
		if (map) {
			map.invalidateSize();
		}
	}
</script>

<svelte:window on:resize={resizeMap} />

<link
	rel="stylesheet"
	href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
	integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
	crossorigin=""
/>
<div class="map" style="height:100%;width:100%" use:mapAction />

<style>
	.map :global(.marker-text) {
		width: 70px;
		padding-left: 0.2em;
		padding-right: 0.2em;
		text-align: center;
		font-weight: 300;
		background-color: rgb(135, 26, 26);
		color: #eee;
		border-radius: 5px;
		white-space: nowrap;
	}

	.map :global(.map-marker) {
		width: 1.5rem;
		transform: translateX(-50%) translateY(-25%);
	}
</style>
