# Vue DeckGL

> Vue wrapper for [DeckGL](https://deck.gl)
_Note: This wrapper let's you render visualization using the base map technique, head over to the deck.gl official documentation to learn more about this._

## Installation

Install the package from NPM. This package requires `@deck.gl/core` to be installed in your current project.

### Install Deck.gl

```bash
npm i @deck.gl/core

```

### Install vue-deck.gl

```bash
npm i vue-deck.gl
```

## Usage

Import the component into your app/components

```js
import VueDeckgl from "vue-deck.gl";
```

### Without a base map

```js
<template>
    <VueDeckgl
        :layers="layers"
        :viewState="viewState"
        @click="handleClick"
        @view-state-change="handleViewStateChange"
    >
    </VueDeckgl>
</template>
<script>
import VueDeckgl from 'vue-deck.gl';

export default {
    components: {
        VueDeckgl
    },
    data() {
        return {
            viewState: {
                latitude: 12.976387,
                longitude: 77.571529,
                zoom: 4,
                bearing: 0,
                pitch: 0,
            },
        }
    },
    methods: {
        handleClick({ event, info }) {
            // handle clicks on the deck instance
        },
        handleViewStateChange(updatedViewState) {
            // update the state object
            this.viewState = {
                ...updatedViewState
            }
        }
    },
    computed: {
        layers() {
            // create layers using @deck.gl/layers
            return [
                //layers
                ]
        }
    }
}
</script>
```

### Using with MapBox as the basemap

```js
<template>
  <VueDeckgl
    :layers="layers"
    :viewState="viewState"
    @click="handleClick"
    @view-state-change="updateViewState"
  >
    <div id="map" ref="map"></div>
  </VueDeckgl>
</template>

<script>
import { PathLayer } from "@deck.gl/layers";
const data = require("../assets/routes-data.json");
import mapboxgl from "mapbox-gl";
import VueDeckgl from 'vue-deck.gl'

export default {
  components: {
    VueDeckgl
  },
  data() {
    return {
      accessToken: process.env.VUE_APP_MAPBOX_TOKEN,
      mapStyle: "mapbox://styles/haxzie/ck0aryyna2lwq1crp7fwpm5vz",
      pathData: data,
      viewState: {
        latitude: 12.976387,
        longitude: 77.571529,
        zoom: 4,
        bearing: 0,
        pitch: 0,
      },
    };
  },
  computed: {
    layers  () {
      const paths = new PathLayer({
        id: "path-layer",
        data: this.pathData,
        widthScale: 20,
        widthMinPixels: 2,
        getPath: (d) => d.path,
        getColor: (d) => [255, (1 - d.data.distance / 100) * 255, 0],
        getWidth: (d) => 2,
      });
      return [paths];
    },
  },
  created() {
    // We need to set mapbox-gl library here in order to use it in template
    this.map = null;
  },
  methods: {
    updateViewState(viewState) {
      console.log("updating view state...");
      this.viewState = {
        ...viewState
      }
      this.map.jumpTo({
        center: [viewState.longitude, viewState.latitude],
        zoom: viewState.zoom,
        bearing: viewState.bearing,
        pitch: viewState.pitch,
      });
    },
    hnadleClick({ event, info }) {

    }
  },
  mounted() {
    // creating the map
    this.map = new mapboxgl.Map({
      accessToken: this.accessToken,
      container: this.$refs.map,
      interactive: false,
      style:
        this.mapStyle || "mapbox://styles/haxzie/ck7h838qb0bik1iofe0k2i3f2",
      center: [this.viewState.longitude, this.viewState.latitude],
      zoom: this.viewState.zoom,
      pitch: this.viewState.pitch,
      bearing: this.viewState.bearing,
    });

    setTimeout(() => {
      const { latitude, longitude, pitch, bearing, zoom } = this.viewState;
      this.viewState = {
        latitude,
        longitude,
        pitch,
        bearing,
        zoom: 12,
        transitionDuration: 3000,
      };
    }, 5000);
  },
};
</script>

<style lang="scss">
#map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #e5e9ec;
  overflow: hidden;
}
</style>

```

### Using with multiple map and deck.gl overlays 
```javascript

<template>
  <div class="deck-container" ref="deckContainerRef">
    <v-deckgl
      :layers="layers"
      :viewState="viewState"
      :disableContextMenu="true"
      :cursor="cursor"
      :controller="{
        doubleClickZoom: false,
        type: MapController,
        scrollZoom: true,
      }"
      @click="(info, event) => $emit('onClick', info, event)"
      @drag="(info, event) => $emit('onDrag', info, event)"
      @onDragStart="(info, event) => $emit('onDragStart', info, event)"
      @onDragEnd="(info, event) => $emit('onDragEnd', info, event)"
      @view-state-change="(viewState) => updateViewState(viewState)"
    >
      <!-- Base map -->
      <template v-slot:background>
        <div id="base-map" ref="map"></div>
      </template>
      <!-- Map Labels -->
      <template v-slot:foreground>
        <div id="foreground-map" ref="fgmap" v-show="showMapLabels"></div>
      </template>
    </v-deckgl>
  </div>
</template>
```