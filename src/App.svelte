<script>
  import {
    Map,
    NavigationControl,
    GeolocateControl,
    Popup,
    Marker,
    LngLatBounds,
  } from "mapbox-gl";
  import { onMount, onDestroy } from "svelte";
  import "../node_modules/mapbox-gl/dist/mapbox-gl.css";

  let map;
  let mapContainer;
  let lng, lat, zoom, pitch, bearing;

  lng = -105.944183;
  lat = 35.691544;
  zoom = 12;
  pitch = 80;
  bearing = 90;

  onMount(() => {
    const initialState = {
      lng: lng,
      lat: lat,
      zoom: zoom,
      pitch: pitch,
      bearing: bearing,
    };

    const dateRanges = [
      "04/15/2024 - 04/26/2024",
      "04/27/2024 - 05/10/2024",
      "05/11/2024 - 05/24/2024",
      "05/25/2024 - 06/01/2024",
      "06/02/2024 - 06/11/2024",
      "06/12/2024 - 07/01/2024",
      "07/02/2024 - 07/08/2024",
      "07/09/2024 - 07/20/2024",
      "07/21/2024 - 09/03/2024",
      "09/04/2024 - 09/05/2024",
      "09/06/2024 - 09/17/2024",
      "09/18/2024 - 10/17/2024",
      "10/18/2024 - 03/19/2025",
    ];

    const waterReleased = 
    [
      0.7,
      1,
      5,
      7.1,
      5,
      3,
      5,
      3,
      2.5,
      1.5,
      1,
      0.6,
      0.3,
    ]

    const dateRangeMappings = {
      uptown: [
        "04/15/2024 - 04/26/2024",
        "09/04/2024 - 09/05/2024",
        "09/06/2024 - 09/17/2024",
        "09/18/2024 - 10/17/2024",
        "10/18/2024 - 03/19/2025",
      ],
      midtown: [
        "04/27/2024 - 05/10/2024",
        "06/12/2024 - 07/01/2024",
        "07/09/2024 - 07/20/2024",
        "07/21/2024 - 09/03/2024",
      ],
      westside: [
        "05/11/2024 - 05/24/2024",
        "05/25/2024 - 06/01/2024",
        "06/02/2024 - 06/11/2024",
        "07/02/2024 - 07/08/2024",
      ],
    };

    map = new Map({
      container: mapContainer,
      accessToken:
        "pk.eyJ1IjoicHNteXRoMTk4NiIsImEiOiJjbHB1bW05bXEwa2RwMmltbzkxOHBtbGFsIn0.8sJLrabsC9aoUdXkZlvdKg",
      style: `mapbox://styles/mapbox/satellite-v9`,
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom,
      pitch: initialState.pitch,
      bearing: initialState.bearing,
    });

    const legendControl = {
      onAdd: function (map) {
        const div = document.createElement("div");
        div.className = "mapboxgl-ctrl";
        div.style.backgroundColor = "#fff";
        div.style.padding = "10px";
        div.innerHTML = `
      <div>
        <div style="background-color: #ff4d00; width: 15px; height: 15px; display: inline-block; margin-right: 5px; border-radius: 50%"></div>
        <span style="font-size: 12px;">Leading Edge Observations</span>
        <br>
        <span style="font-size: 10px;">(Click on the points to view details)</span>
        <br>
        <div style="background-color: blue; width: 30px; height: 10px; display: inline-block; margin-right: 5px; border-radius: 0%"></div>
        <span style="font-size: 12px;">Santa Fe River</span>
        <br>
        <span style="font-size: 10px;">(River reach can be filtered using date range above)</span>
      </div>
    `;

        return div;
      },

      onRemove: function () {
        // You can add code here to handle the removal of the control if needed
      },

      getDefaultPosition: function () {
        return "bottom-left";
      },
    };

    map.addControl(legendControl);

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
      map.setTerrain({ source: "mapbox-dem", exaggeration: 2 });

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
          "line-width": 5,
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
          "line-width": 5,
          "line-dasharray": [0, 4, 3],
        },
      });

      // Add a new source from our ArcGIS Feature Server API
      map.addSource("points", {
        type: "geojson",
        data: "https://services8.arcgis.com/5PqKWBI6rgHxYkkd/arcgis/rest/services/survey123_1320ecf6756941bb9c1ae5683cb0bde3_results/FeatureServer/0/query?where=1%3D1&outFields=*&outSR=4326&f=geojson",
      });

      // Add a new layer to use the source
      map.addLayer({
        id: "points",
        type: "circle",
        source: "points",
        paint: {
          "circle-radius": 4,
          "circle-color": "#ff4d00",
          "circle-stroke-color": "black",
          "circle-stroke-width": 1,
        },
      });

      const calloutFeatures = {
        type: "FeatureCollection",
        features: [
          {
            type: "Feature",
            properties: {
              order: 1,
              name: "Nichols Reservoir",
              height: 2200,
            },
            geometry: {
              type: "Point",
              coordinates: [-105.8802, 35.6905],
            },
          },
          {
            type: "Feature",
            properties: {
              order: 2,
              name: "St. Francis",
              height: 2000,
            },
            geometry: {
              type: "Point",
              coordinates: [-105.95404777099925, 35.688445942312825],
            },
          },
          {
            type: "Feature",
            properties: {
              order: 3,
              name: "Frenchy's Field",
              height: 1900,
            },
            geometry: {
              type: "Point",
              coordinates: [-105.9851838550692, 35.67352519263433],
            },
          },
          {
            type: "Feature",
            properties: {
              order: 4,
              name: "San Isidro Crossing",
              height: 1800,
            },
            geometry: {
              type: "Point",
              coordinates: [-106.01218164975715, 35.65996195043828],
            },
          },
          {
            type: "Feature",
            properties: {
              order: 4,
              name: "Water Treatment Facility",
              height: 1700,
            },
            geometry: {
              type: "Point",
              coordinates: [-106.08782667247951, 35.632331143214415],
            },
          },
        ],
      };

      for (const marker of calloutFeatures.features) {
        // Create a DOM element for each marker.
        const el = document.createElement("div");
        el.className = "marker";
        const size = marker.properties.height / 100;
        el.style.width = `${size}px`;
        el.style.height = `${size}px`;

        const popup = new Popup({ offset: 25 }).setText(marker.properties.name);

        // Add markers to the map.
        new Marker({
          element: el,
          // Point markers toward the nearest horizon
          rotationAlignment: "horizon",
          offset: [0, -size / 2],
        })
          .setLngLat(marker.geometry.coordinates)
          .setPopup(popup) // associate the popup with the marker
          .addTo(map);

        //add the popup to the map
        popup.addTo(map);
      }

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

    map.on("click", "points", function (e) {
      var coordinates = e.features[0].geometry.coordinates.slice();
      var properties = e.features[0].properties;

      var displayProperties = [
        "date_and_time",
        "observer_name",
        "notes_about_observation",
      ];

      // Create an HTML string for the popup
      var description = Object.keys(properties)
        .filter(function (key) {
          return displayProperties.includes(key);
        })
        .map(function (key) {
          var value = properties[key];
          if (key === "date_and_time") {
            value = new Date(value).toLocaleString();
          }
          return `<strong>${key}:</strong> ${value}`;
        })
        .join("<br>");

      // Ensure that if the map is zoomed out such that multiple
      // copies of the feature are visible, the popup appears
      // over the copy being pointed to.
      while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
        coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
      }

      new Popup().setLngLat(coordinates).setHTML(description).addTo(map);
    });

    // Change the cursor to a pointer when the mouse is over the points layer.
    map.on("mouseenter", "points", function () {
      map.getCanvas().style.cursor = "pointer";
    });

    // Change it back to a pointer when it leaves.
    map.on("mouseleave", "points", function () {
      map.getCanvas().style.cursor = "";
    });

    map.on("load", () => {
      const filterGroup = document.getElementById("filter-group");

      // this function will be called whenever a checkbox is toggled
      const updateLayerFilter = () => {
        const checkedSymbols = [...document.getElementsByTagName("input")]
          .filter((el) => el.checked)
          .map((el) => el.id);

        let reach = [];

        function convertDateRanges(dateRanges) {
          return dateRanges.map((range) => {
            const [start, end] = range
              .split(" - ")
              .map((dateStr) => new Date(dateStr));
            return { start, end };
          });
        }

        function convertToISODate(dateStr) {
          const date = new Date(dateStr);
          return date;
        }

        function isDateInRange(date, ranges) {
          date = convertToISODate(date);

          return ranges.some(
            (range) => date >= range.start && date <= range.end
          );
        }

        // Retrieve all features from the "points" source
        const obs = map.querySourceFeatures("points");

        // Further filter features based on date range
        const finalFilteredFeatures = obs.filter((feature) => {
          const date = new Date(
            feature.properties["date_and_time"]
          ).toLocaleDateString();
          return isDateInRange(date, convertDateRanges(checkedSymbols));
        });

        // Get the IDs of the final filtered features
        let finalFilteredFeatureIds = finalFilteredFeatures.map(
          (feature) => feature.id
        );

        //if finalFilteredFeatureIds is empty, then get array of all feature ids
        if (checkedSymbols.length === 0) {
          finalFilteredFeatureIds = obs.map((feature) => feature.id);
        }

        //if checkedSymbol is in the dateRangeMappings object, then use the value of the ke
        for (const [key, value] of Object.entries(dateRangeMappings)) {
          if (value.includes(checkedSymbols[0])) {
            reach.push(key);
          }
        }

        if (reach.includes("uptown")) {
          reach = ["uptown"];
        } else if (reach.includes("midtown")) {
          reach = ["uptown", "midtown"];
        } else {
          reach = ["uptown", "midtown", "westside"];
        }

        // use an 'in' expression to filter the layer
        map.setFilter("route", ["in", "reach", ...reach]);
        map.setFilter("route-dashed", ["in", "reach", ...reach]);
        map.setFilter("points", ["in", "objectid", ...finalFilteredFeatureIds]);

        const zoomPoints = map.querySourceFeatures("points", {
          filter: ["in", "objectid", ...finalFilteredFeatureIds],
        });

        if (zoomPoints.length) {
          const bounds = new LngLatBounds();

          zoomPoints.forEach((feature) => {
            bounds.extend(feature.geometry.coordinates);
          });

          map.fitBounds(bounds, {
            padding: 20,
          });
        }
      };

      // for each `range` value, create a checkbox and label
      for (const range of dateRanges) {
        const input = document.createElement("input");
        input.type = "checkbox";
        input.id = range;
        input.checked = false;
        filterGroup.appendChild(input);

        const label = document.createElement("label");
        label.setAttribute("for", range);
        label.textContent = range;
        filterGroup.appendChild(label);

        //add values from waterReleased array to end of the label and append 'cfs'
        const waterReleasedValue = document.createElement("span");
        waterReleasedValue.textContent = ` - ${waterReleased[dateRanges.indexOf(range)]} cfs`;
        label.appendChild(waterReleasedValue);

        //add info icon to end of the label that has tooltip text that read "Planned flow release volume"
        const infoIcon = document.createElement("span");
        infoIcon.className = "info-icon";
        infoIcon.title = "Planned flow release volume";
        infoIcon.textContent = "ℹ️";
        label.appendChild(infoIcon);

        // When any checkbox changes, update the filter.
        input.addEventListener("change", (event) => {
          // Uncheck all other checkboxes
          const checkboxes = filterGroup.querySelectorAll(
            "input[type='checkbox']"
          );
          checkboxes.forEach((checkbox) => {
            if (checkbox !== event.target) {
              checkbox.checked = false;
            }
          });

          // Update the filter
          updateLayerFilter();
        });
      }
    });
  });

  function resetMapView() {
    map.flyTo({
      center: [initialState.lng, initialState.lat],
      zoom: initialState.zoom,
      pitch: initialState.pitch,
      bearing: initialState.bearing,
    });
  }

  onDestroy(() => {
    map.remove();
  });
</script>

<head> </head>

<div>
  <header id="header-group" class="header-group">
    2024-2025 Santa Fe Living River Flow Release Schedule <span
      class="info-icon"
      title="Planned flow releases for the Santa Fe Living River."
    >
      ℹ️
    </span>
  </header>
  <nav id="filter-group" class="filter-group"></nav>
  <div class="map-wrap">
    <div class="map" bind:this={mapContainer} />
  </div>
  <!-- <button id="reset-button" on:click={resetMapView}>Reset View</button> -->
</div>

<style>
  .map {
    position: absolute;
    width: 100%;
    height: 100%;
  }
</style>
