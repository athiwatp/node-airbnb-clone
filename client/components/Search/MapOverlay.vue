<template>

  <div class="map-overlay">
    <div class="map-tooltip">
      <slot name="marker"></slot>
    </div>
    <div class="map-dialog">
      <slot name="tooltip"></slot>
    </div>
  </div>

</template>

<script>
  import RichMarker from 'googlemaps-js-rich-marker';
  import $ from 'jquery';
  import _ from 'lodash';

  var infoWindows = [];

  export default {
    name: 'map-overlay',
    props: ['map', 'overlay'],
    localData: {
    },
    data() {
      return {
        marker: null,
        infoWindow: null
      }
    },
    methods: {
      setOverlay(overlay, oldOverlay) {
        var elem = $(this.$el);
        var latlng = overlay.position;

        if (oldOverlay && overlay.id === oldOverlay.id && this.marker) {
          var tooltipContent = elem.find('.map-tooltip');
          tooltipContent.toggleClass('selected', typeof(overlay.selected) !== 'undefined' && overlay.selected)
          this.marker.setContent(tooltipContent[0].outerHTML)

          return;
        }

        if (this.marker) {
          this.marker.setMap(null);
        }

        var infoContent = elem.find('.map-dialog');
        var tooltipContent = elem.find('.map-tooltip');
        tooltipContent.toggleClass('selected', overlay.selected || false)

        this.infoWindow = new google.maps.InfoWindow({
          content: infoContent[0].outerHTML,
          pixelOffset: new google.maps.Size(0, -20)
        });

        var marker = new RichMarker.RichMarker({
          map: this.map,
          shadow: 'none',
          position: new google.maps.LatLng(latlng.lat, latlng.lng),
          content: tooltipContent[0].outerHTML
        });

        google.maps.event.addListener(marker, 'click', (event) => {
          this.infoWindow.open(this.map, marker);

          infoWindows.forEach((i) => {
            if (i.anchor != this.marker)
              i.close();
          });
          infoWindows.length = 0;
          infoWindows.push(this.infoWindow);
        });

        google.maps.event.addListener(marker, 'mouseover', () => {
          this.$store.commit('setOfferSelected', overlay.id);
        });
        google.maps.event.addListener(marker, 'mouseout', () => {
          this.$store.commit('setOfferUnselected', overlay.id);
        });

        this.marker = marker;
      }
    },
    created: function () {
    },
    watch: {
      map: function (map) {
        this.setOverlay(this.overlay);
      },
      overlay(newOverlay, oldValue) {
        this.setOverlay(newOverlay, oldValue);
      }
    },
  };

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .map-tooltip {
    background-color: red;
    color: white;
    padding: 5px;
    pointer-events: none;
  }
  
  .map-tooltip.selected {
    background-color: green;
  }
  
  .map-tooltip:hover {
    cursor: pointer;
  }
</style>