<script>
import { Deck, FlyToInterpolator } from "@deck.gl/core";

export default {
  name: "VueDeckgl", // vue component name
  props: {
    layers: Array,
    viewState: Object,
  },
  methods: {
    /**
     * Initializes a new deck.gl instance with specified initialViewState and Layers
     * @param {Object} initialViewState of the deckGL
     * @param {Array} layers to be renderd
     */
    initializeDeck(viewState, layers) {
      return new Deck({
        canvas: this.$refs.canvas,
        width: "100%",
        height: "100%",
        viewState,
        controller: true,
        // change the map's viewstate whenever the view state of deck.gl changes
        onViewStateChange: ({ viewState }) => {
          this.deck.setProps({
            viewState,
          });
          this.$emit("view-state-change", viewState);
        },
        // emit click events back to parent
        onClick: (event, info) => {
          this.$emit("click", { event, info });
        },
        layers: layers,
      });
    },
  },
  watch: {
    /**
     * Whenever the layer prop changes,update the layers of deck.gl
     */
    layers(newLayers) {
      this.deck.setProps({
        layers: newLayers,
      });
    },
    /**
     * Update the viewstate of deckgl whenever the prop changes
     */
    viewState(newViewState) {
      this.deck.setProps({
        viewState: {
          ...newViewState,
          // add flyto transitions whenever the viewstates change
          // additionally this to work, the user needs to specify the transitionDuration
          // within the newViewState
          transitionInterpolator: new FlyToInterpolator(),
        },
      });
    },
  },
  created() {
    // needs to be a non-reactive element
    this.deck = null;
  },
  mounted() {
    this.deck = this.initializeDeck(this.viewState, this.layers);
  },
  destroyed() {
    this.deck.finalize();
  },
};
</script>

<template>
  <div class="deckgl-container">
    <slot></slot>
    <canvas id="deck-canvas" ref="canvas"></canvas>
  </div>
</template>

<style scoped>
.deck-container {
  width: 100%;
  height: 100%;
  position: relative;
}

.deck-container #map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #e5e9ec;
  overflow: hidden;
}

.deck-container #deck-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
