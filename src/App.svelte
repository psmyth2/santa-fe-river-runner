<script>
  import { Map, NavigationControl, GeolocateControl } from "mapbox-gl";
  import { onMount, onDestroy } from "svelte";
  import "../node_modules/mapbox-gl/dist/mapbox-gl.css";

  let map;
  let mapContainer;
  let lng, lat, zoom, pitch, bearing;

  lng = -105.944183;
  lat = 35.691544;
  zoom = 13;
  pitch = 75;
  bearing = 65;

  onMount(() => {
    const initialState = {
      lng: lng,
      lat: lat,
      zoom: zoom,
      pitch: pitch,
      bearing: bearing,
    };

    map = new Map({
      container: mapContainer,
      accessToken:
        "pk.eyJ1IjoicHNteXRoMTk4NiIsImEiOiJjbHB1bW05bXEwa2RwMmltbzkxOHBtbGFsIn0.8sJLrabsC9aoUdXkZlvdKg",
      style: `mapbox://styles/mapbox/light-v10`,
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom,
      pitch: initialState.pitch,
      bearing: initialState.bearing,
    });

    map.addControl(new NavigationControl());
    map.addControl(new GeolocateControl());

    map.on("style.load", () => {
      map.addSource("mapbox-dem", {
        type: "raster-dem",
        url: "mapbox://mapbox.mapbox-terrain-dem-v1",
        tileSize: 512,
        maxzoom: 13,
      });
      // add the DEM source as a terrain layer with exaggerated height
      map.setTerrain({ source: "mapbox-dem", exaggeration: 1.5 });

      //add a sky layer that will show when the map is highly pitched
      map.addLayer({
        id: "sky",
        type: "sky",
        paint: {
          "sky-type": "atmosphere",
          "sky-atmosphere-color": "rgb(186, 210, 235)",
          "sky-atmosphere-sun": [0.0, 0.0],
          "sky-atmosphere-halo-color": "rgb(255, 87, 51)",
          "sky-atmosphere-sun-intensity": 8,
        },
      });

      //add source
      map.addSource("river", {
        type: "geojson",
        data: "santafe_river_reaches.geojson",
      });

      // Add styles to the map
      map.addLayer({
        id: "route",
        type: "line",
        source: "river",
        paint: {
          "line-color": "blue",
          "line-width": 4,
          "line-opacity": 0.4,
        },
      });

      // Add styles to the map
      map.addLayer({
        id: "route-dashed",
        type: "line",
        source: "river",
        paint: {
          "line-color": "blue",
          "line-width": 4,
          "line-dasharray": [0, 4, 3],
        },
      });

      // an array of valid line-dasharray values, specifying the lengths of the 
      //alternating dashes and gaps that form the dash pattern

      const dashArraySequence = [
        [0, 4, 3],
        [0.5, 4, 2.5],
        [1, 4, 2],
        [1.5, 4, 1.5],
        [2, 4, 1],
        [2.5, 4, 0.5],
        [3, 4, 0],
        [0, 0.5, 3, 3.5],
        [0, 1, 3, 3],
        [0, 1.5, 3, 2.5],
        [0, 2, 3, 2],
        [0, 2.5, 3, 1.5],
        [0, 3, 3, 1],
        [0, 3.5, 3, 0.5],
      ];

      let step = 0;

      function animateDashArray(timestamp) {
        // Update line-dasharray using the next value in dashArraySequence. The
        // divisor in the expression `timestamp / 50` controls the animation speed.
        const newStep = parseInt((timestamp / 50) % dashArraySequence.length);

        if (newStep !== step) {
          map.setPaintProperty(
            "route-dashed",
            "line-dasharray",
            dashArraySequence[step]
          );
          step = newStep;
        }

        // Request the next frame of the animation.
        requestAnimationFrame(animateDashArray);
      }

      // start the animation
      animateDashArray(0);
    });

    map.on("load", () => {
	  const santafe_river_reaches = map.querySourceFeatures("river");

      const filterGroup = document.getElementById("filter-group");

      // this function will be called whenever a checkbox is toggled
      const updateLayerFilter = () => {
        // get an array of icon names that corresponds to the currently checked checkboxes
        const checkedSymbols = [...document.getElementsByTagName("input")]
          .filter((el) => el.checked)
          .map((el) => el.id);

        // use an 'in' expression to filter the layer
        map.setFilter("route", ["in", "reach", ...checkedSymbols]);
        map.setFilter("route-dashed", ["in", "reach", ...checkedSymbols]);
      };

      // get an array of all unique `reach` properties
      const reaches = [];

      for (const feature of santafe_river_reaches) {
        const reach = feature.properties.reach;
        if (!reaches.includes(reach)) reaches.push(reach);
      }

      // for each `reach` value, create a checkbox and label
      for (const reach of ['uptown', 'midtown', 'westside']) {
        const input = document.createElement("input");
        input.type = "checkbox";
        input.id = reach;
        input.checked = true;
        filterGroup.appendChild(input);

        const label = document.createElement("label");
        label.setAttribute("for", reach);
        label.textContent = reach;
        filterGroup.appendChild(label);

        // When any checkbox changes, update the filter.
        input.addEventListener("change", updateLayerFilter);
      }
    });
  });

  onDestroy(() => {
    map.remove();
  });
</script>

<head> </head>

<div>
  <nav id="filter-group" class="filter-group"></nav>
  <div class="map-wrap">
    <div class="map" bind:this={mapContainer} />
  </div>
</div>

<style>
  .map {
    position: absolute;
    width: 100%;
    height: 100%;
  }

</style>
