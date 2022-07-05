<script>
  import { onMount } from "svelte";
  import GooglePlacesAutocomplete from "@silintl/svelte-google-places-autocomplete";
  import RangeSlider from "svelte-range-slider-pips";
  import "./lib/Tailwind.css";
  import Map from "./lib/Map.svelte";
  import mapStyles from "./lib/map-styles";

  let map,
    marker,
    circleRadius,
    radius,
    address = "",
    lat,
    lng;

  const defaultRadius = 20;
  const map_canvas = document.getElementById("map-canvas");

  // init google maps
  onMount(async () => {
    let google = window.google;
    
    const mapOptions = {
      zoom: 12,
    };

    map = new google.maps.Map(
      document.getElementById("map-canvas"),
      mapOptions
    );

    // Get GEOLOCATION
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        function (position) {
          lat = position.coords.latitude;
          lng = position.coords.longitude;
          console.log("Latitude:", lat);
          console.log("Longitude:", lng);

          let pos = new google.maps.LatLng(lat, lng);
          
          map.setCenter(pos);

          marker = new google.maps.Marker({
            position: pos,
            map: map,
            draggable: true,
          });

          radius = defaultRadius;
          console.log("Radius:", radius);
          circleRadius = defaultRadius * 1000;
          marker.Circle = new google.maps.Circle({
            center: marker.getPosition(),
            strokeColor: "#3B82F6",
            strokeOpacity: 0.8,
            strokeWeight: 2,
            fillColor: "#3B82F6",
            fillOpacity: 0.35,
            radius: 6500,
            map: map,
            editable: false,
          });

          marker.Circle.bindTo("center", marker, "position");
        },
        function () {
          handleNoGeolocation(true);
        }
      );
    } else {
      handleNoGeolocation(false);
    }
  });

  // handle geo location
  function handleNoGeolocation(errorFlag) {
    if (errorFlag) {
      let content = "Error: The Geolocaiton service failed";
    } else {
      let content = "Error: Your browser doesn't support geolocation.";
    }

    lat = map_canvas.getAttribute("data-lat");
    lng = map_canvas.getAttribute("data-lng");
    console.log("Latitude:", lat);
    console.log("Longitude:", lng);

    let options = {
      map: Map,
      position: new google.maps.LatLng(lat, lng),
      content: content,
    };

    mapStyles.setCenter(options.position);
    marker = new google.maps.Marker({
      position: options.position,
      map: mapStyles,
      draggable: true,
    });

    radius = defaultRadius;
    console.log("Radius:", radius);
    circleRadius = defaultRadius * 1000;
    marker.Circle = new google.maps.Circle({
      center: marker.getPosition(),
      strokeColor: "#3B82F6",
      strokeOpacity: 0.8,
      strokeWeight: 2,
      fillColor: "#3B82F6",
      fillOpacity: 0.35,
      radius: 6500,
      map: map,
      editable: false,
    });

    marker.Circle.bindTo("center", marker, "position");
  }

  // google auto complete
  const autocomplete_options = {
    fields: ["name", "icon", "photos", "place_id", "geometry", "type", "url"],
    types: ["establishment"],
  };

  const onAutoCompletePlaceChanged = (ev) => {
    marker.setVisible(false);
    let place = ev.detail.place;
    if (!place.geometry) {
      return;
    }

    // If the place has a geometry, then present it on a map.
    if (place.geometry.viewport) {
      map.fitBounds(place.geometry.viewport);
    } else {
      map.setCenter(place.geometry.location);
      map.setZoom(12);
    }
    marker.setIcon(
      /** @type {google.maps.Icon} */ ({
        url: place.icon,
        size: new google.maps.Size(71, 71),
        origin: new google.maps.Point(0, 0),
        anchor: new google.maps.Point(17, 34),
        scaledSize: new google.maps.Size(35, 35),
      })
    );

    lat = place.geometry.location.lat();
    lat = place.geometry.location.lng();
    console.log("Latitude:", lat);
    console.log("Longitude:", lng);

    marker.setPosition(place.geometry.location);
    marker.setVisible(true);

    address = "";
    if (place.address_components) {
      address = [
        (place.address_components[0] &&
          place.address_components[0].short_name) ||
          "",
        (place.address_components[1] &&
          place.address_components[1].short_name) ||
          "",
        (place.address_components[2] &&
          place.address_components[2].short_name) ||
          "",
      ].join(" ");
    }
    console.log("Address:", address)
  };

  // change range slider
  const onHandleSlider = (ev) => {
    radius = ev.detail.value;
    circleRadius = radius * 1000;
    console.log("Radius:", radius);

    marker.Circle.setMap(null);
    marker.Circle = new google.maps.Circle({
      center: marker.getPosition(),
      strokeColor: "#3B82F6",
      strokeOpacity: 0.8,
      strokeWeight: 2,
      fillColor: "#3B82F6",
      fillOpacity: 0.35,
      radius: circleRadius,
      map: map,
      editable: false,
    });

    marker.Circle.bindTo("center", marker, "position");
  };
</script>

<section class="text-gray-600 body-font">
  <div
    class="container mx-auto flex px-5 py-8 items-center justify-center flex-col"
  >
    <div class="text-center lg:w-2/3 w-full">
      <h1
        class="title-font sm:text-4xl text-3xl mb-4 font-medium text-gray-900"
      >
        Svelte + vite Google maps api
      </h1>
      <div class="grid grid-cols-2 gap-6 my-8">
        <div>
          <div class="my-4">
            <label
              class="block text-gray-700 text-md font-bold mb-2 text-left"
              for=""
            >
              Advertise near an address
            </label>
            <GooglePlacesAutocomplete
              apiKey="AIzaSyB5HYrOwNUgeMxMWUx3QGp8fev-PWjFoYw"
              class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              on:place_changed={onAutoCompletePlaceChanged}
              {autocomplete_options}
              placeholder="Enter address"
              value=""
            />
          </div>
          <div class="mt-12 range-slider">
            <RangeSlider
              values={[defaultRadius]}
              min={5}
              max={65}
              step={5}
              hover={true}
              float
              suffix="km"
              pips
              on:change={onHandleSlider}
            />
          </div>
          <div class="mt-12">
            <ul
              class="list-disc list-inside text-slate-700 bg-gray-200 rounded-xl text-left py-5 px-5"
            >
              <li class="text-md mb-1 text-left">
                Address: <span class="text-sky-500">{address}</span>
              </li>
              <li class="text-md mb-1 text-left">
                Latitude: <span class="text-sky-500">{lat}</span>
              </li>
              <li class="text-md mb-1 text-left">
                Longitude: <span class="text-sky-500">{lng}</span>
              </li>
              <li class="text-md mb-1 text-left">
                Radius: <span class="text-sky-500">{radius}km</span>
              </li>
            </ul>
          </div>
        </div>
        <div>
          <div
            id="map-canvas"
            class="relative w-full h-full rounded"
            data-lat="40.748817"
            data-lng="-73.985428"
          />
        </div>
      </div>
    </div>
  </div>
</section>

<style>
  .range-slider {
    margin-left: -1rem;
    margin-right: -1rem;
  }
  #map-canvas {
    min-height: 30rem;
  }
</style>
