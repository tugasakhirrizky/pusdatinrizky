<!DOCTYPE html>
<html lang="en">
<head>
  <title> Pusdatin </title>
  <meta property="og:description" content="Use mapbox-gl-draw to draw a polygon and Turf.js to calculate its area in square meters." />
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Display a Geocoding Control</title>
  <script src="https://cdn.maptiler.com/maptiler-sdk-js/v1.2.0/maptiler-sdk.umd.min.js"></script>
  <link href="https://cdn.maptiler.com/maptiler-sdk-js/v1.2.0/maptiler-sdk.css" rel="stylesheet" />
  <link rel='stylesheet' href='https://unpkg.com/maplibre-gl@3.6.2/dist/maplibre-gl.css' />
  <script src="https://cdn.maptiler.com/maptiler-geocoding-control/v1.2.0/maptilersdk.umd.js"></script>
  <link href="https://cdn.maptiler.com/maptiler-geocoding-control/v1.2.0/style.css" rel="stylesheet">
  <script src='https://unpkg.com/maplibre-gl@3.6.2/dist/maplibre-gl.js'></script>
  <style>
    body { margin: 0; padding: 0; font-family: sans-serif; }
    #map { position: absolute; top: 0; bottom: 0; width: 100%; }
  </style>
</head>
<body>
<style>
     .calculation-box {
        height: 75px;
        width: 150px;
        position: absolute;
        bottom: 40px;
        left: 10px;
        background-color: rgba(255, 255, 255, 0.9);
        padding: 15px;
        text-align: center;
    }

    p {
        font-family: 'Open Sans';
        margin: 0;
        font-size: 13px;
    }
    #file {
        position: absolute;
        bottom: 150px;
        left: 50px;
    }
</style>
  <div id="map"></div>
  <input 
        type="file"
        id="file"
        name="file"
        accept="application/geo+json,application/vnd.geo+json,.geojson"
    />
  <div class="calculation-box">
        <p> Draw a polygon using the draw tools.</p>
        <div id="calculated-area"></div>
  <script>

      maptilersdk.config.apiKey = 'nf9GSRIgi442J9YzDFhG';

      const map = (window.map = new maptilersdk.Map({
        container: 'map', // container's id or the HTML element to render the map
        style: 'https://api.maptiler.com/maps/streets/style.json?key=nf9GSRIgi442J9YzDFhG',  // stylesheet location
        zoom: 3,
        center: [-94.77, 38.57],
      }));

      map.on('load', function () {
        map.addSource('wms-test-source', {
            'type': 'raster',
            // use the tiles option to specify a WMS tile source URL
            // https://docs.maptiler.com/gl-style-specification/sources/
            'tiles': [
                'https://img.nj.gov/imagerywms/Natural2015?bbox={bbox-epsg-3857}&format=image/png&service=WMS&version=1.1.1&request=GetMap&srs=EPSG:3857&transparent=true&width=256&height=256&layers=Natural2015'
            ],
            'tileSize': 256
        });
        map.addLayer(
            {
                'id': 'wms-test-layer',
                'type': 'raster',
                'source': 'wms-test-source',
                'paint': {}
            },
            'aeroway_fill'
        );
     });
        
      function handleFileSelect(evt) {
        const file = evt.target.files[0]; // Read first selected file

        const reader = new FileReader();

        reader.onload = function (theFile) {
            // Parse as (geo)JSON
            const GeoJSONcontent = JSON.parse(theFile.target.result);

            // Add as source to the map
            map.addSource('uploaded-source', {
                'type': 'geojson',
                'data': GeoJSONcontent
            });

            map.addLayer({
                'id': 'uploaded-polygons',
                'type': 'fill',
                'source': 'uploaded-source',
                'paint': {
                    'fill-color': '#DFFF00',
                    'fill-outline-color': 'red',
                    'fill-opacity': 0.7
                },
                // filter for (multi)polygons; for also displaying linestring
                // or points add more layers with different filters
                'filter': ['==', '$type', 'Polygon']
            });
            
            // When a click event occurs on a feature in the states layer, open a popup at the
            // location of the click, with description HTML from its properties.
            map.on('click', 'uploaded-polygons', (e) => {
                var p = e.features[0].properties;
                
                new maplibregl.Popup()
                .setLngLat(e.lngLat)
                .setHTML(`<p> <b> Nomor Identifikasi Bidang: </b> ${p.NIB} </p> <p> <b> Nama: </b> ${p.Nama} </p> <p> <b> Luas Tanah: </b>  ${p.Luas} </p>`)
                .addTo(map);
            });

            // Change the cursor to a pointer when the mouse is over the states layer.
            map.on('mouseenter', 'uploaded-polygons', () => {
                map.getCanvas().style.cursor = 'pointer';
            });

            // Change it back to a pointer when it leaves.
            map.on('mouseleave', 'uploaded-polygons', () => {
                map.getCanvas().style.cursor = '';
            });
        };

        // Read the GeoJSON as Text
        reader.readAsText(file, 'UTF-8');
      }

      document
        .getElementById('file')
        .addEventListener('change', handleFileSelect, false);
        
      const gc = new maptilersdkMaptilerGeocoder.GeocodingControl({});

      map.addControl(gc, 'top-left');

    </script>
<style>
    .calculation-box {
        height: 75px;
        width: 150px;
        position: absolute;
        bottom: 40px;
        left: 10px;
        background-color: rgba(255, 255, 255, 0.9);
        padding: 15px;
        text-align: center;
    }

    p {
        font-family: 'Open Sans';
        margin: 0;
        font-size: 13px;
    }
</style>
<script src="https://api.tiles.mapbox.com/mapbox.js/plugins/turf/v3.0.11/turf.min.js"></script>
<script src="https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-draw/v1.4.2/mapbox-gl-draw.js"></script>
<link
    rel="stylesheet"
    href="https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-draw/v1.2.0/mapbox-gl-draw.css"
    type="text/css"
/>

<script>
    MapboxDraw.constants.classes.CONTROL_BASE  = 'maplibregl-ctrl';
    MapboxDraw.constants.classes.CONTROL_PREFIX = 'maplibregl-ctrl-';
    MapboxDraw.constants.classes.CONTROL_GROUP = 'maplibregl-ctrl-group';

    const draw = new MapboxDraw({
        displayControlsDefault: false,
        controls: {
            polygon: true,
            trash: true
        }
    });
    map.addControl(draw, 'bottom-right');

    map.on('draw.create', updateArea);
    map.on('draw.delete', updateArea);
    map.on('draw.update', updateArea);

    function updateArea(e) {
        const data = draw.getAll();
        const answer = document.getElementById('calculated-area');
        if (data.features.length > 0) {
            const area = turf.area(data);
            // restrict to area to 2 decimal points
            const roundedArea = Math.round(area * 100) / 100;
            answer.innerHTML =
                `<p><strong>${
                    roundedArea
                }</strong></p><p>square meters</p>`;
        } else {
            answer.innerHTML = '';
            if (e.type !== 'draw.delete')
                alert('Use the draw tools to draw a polygon!');
        }
    };
</script>
</body>
</html>
