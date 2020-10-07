
<template>
  <section id="main">

    <div class="container is-fluid">
    	<div class="box">

        <div class="field has-addons">
  <div class="control is-expanded">
    <input class="input" type="text" v-model="objectUrl" placeholder="Enter a Digital Collections or Digital Commonwealth URL">
  </div>
  <div class="control">
    <a class="button is-primary" @click="resolveObjectUrl">
      Open this URL
    </a>
  </div>
</div>

        

      </div>
    </div>
  
    <div class="container is-fluid" :class="[error ? '' : 'is-hidden']" id="error">
      <div class="notification is-warning is-light">{{ errorMessage }}</div>
    </div>

    <div class="container is-fluid" :class="[manifestLoaded ? '' : 'is-hidden']">
      <div class="box">

        <h2 class="is-size-4">Object Information</h2>

        <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      Digital Commonwealth URL 
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="dcObjectUrl">
  </p>

</div>

        <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      LMEC Collections URL
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="lmecObjectUrl">
  </p>

</div>

        <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      ARK Permalink
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="arkLink">
  </p>

</div>

        <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      IIIF Manifest
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="manifestUrl">
  </p>

</div>
      </div>
    </div>

    <div class="container is-fluid" :class="[manifestLoaded ? '' : 'is-hidden']">
      <div class="box">
            <h2 class="is-size-4">Images</h2>
      <ul>
        <li v-for="(sequence, seqIndex) in manifest.sequences">Sequence {{seqIndex}}
          <ul>
            <li v-for="(canvas, canIndex) in sequence.canvases">Canvas {{canIndex}} ({{canvas.height}} ×{{canvas.width}})
              
              <ul>
                <li v-for="(image, imgIndex) in canvas.images">Image {{imgIndex}} <button class="button is-small is-primary" @click="loadImage(seqIndex,canIndex,imgIndex)">Load into viewer</button></li>
                </ul>

            </li>

            </ul>
        </li>
      </ul>
      </div>
  
    </div>

    <div class="container is-fluid" :class="[state === 'imageLoaded' ? '' : 'is-hidden']">
      <div class="box">
        <h2 class="is-size-4">Sequence {{ this.loadedImg[0] }}, Canvas {{this.loadedImg[1] }}, Image {{this.loadedImg[2] }}</h2>

                <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      IIIF Image Endpoint
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="loadedImgIIIF">
  </p>
  </div>

  <div class="field is-horizontal">
  <div class="field-label is-normal">
    <label class="label">Image Quality</label>
  </div>
  <div class="field-body">
    <div class="field is-narrow">
      <div class="control">
        <div class="select is-fullwidth">
          <select v-model="imgType">
                    <option>default</option>
                    <option>color</option>
                    <option>gray</option>
                    <option>bitonal</option>
                  </select>
        </div>
      </div>
    </div>
  </div>
</div>

                <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      Full Image
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="loadedImgIIIF + '/full/full/0/' + imgType + '.jpg'">
  </p>
  </div>

  </div></div>
   <div class="container is-fluid" :class="[state === 'imageLoaded' ? '' : 'is-invisible']">
      <div class="box">


                <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      Extent Coordinate Array
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="extentCoords">
  </p>
  </div>

                  <div class="field has-addons">
            <p class="control">
    <a class="button is-static">
      Extent Image
    </a>
  </p>
  <p class="control is-expanded">
    <input class="input" type="text" readonly :value="loadedImgIIIF + '/' + Math.round(extentCoords[0]) + ',' + Math.round(extentCoords[3]*-1) + ',' + Math.round(extentCoords[2]-extentCoords[0]) + ',' + Math.round(extentCoords[1]*-1 - extentCoords[3]*-1) + '/full/0/' + imgType + '.jpg'">
  </p>
  </div>

          <div id="ol-map"></div>


          </div>
        </div>

    </section>
</template>

<script lang="ts">
import Vue from "vue";

import Map from 'ol/Map.js';
import View from 'ol/View.js';
import TileLayer from 'ol/layer/Tile.js'  
import OSM from "ol/source/OSM"
import IIIF from 'ol/source/IIIF';
import IIIFInfo from 'ol/format/IIIFInfo';
import ExtentInteraction from 'ol/interaction/Extent';

  export default Vue.extend({

    mounted() {
      this.layer = new TileLayer();
      this.map = new Map({
        target: 'ol-map',
        layers: [
          this.layer
        ],
        view: new View({
          center: [0, 0],
          zoom: 2
        })
      });
      this.extent = new ExtentInteraction();
      this.map.addInteraction(this.extent);
      this.extent.setActive(false);

      window.addEventListener('keydown', (event) => {
      if (event.keyCode == 16) {
      this.extent.setActive(true);
      }

      window.addEventListener('keyup', (event) => {
      if (event.keyCode == 16) {
      this.extent.setActive(false);
      }
      
      this.extent.on('extentchanged', (e) => { this.$data.extentCoords = this.extent.getExtent(); });

})
});
    },

    data: function() {
      return {
        state: "initial",
        error: false,
        errorMessage: '',
        manifestLoaded: false,
        objectUrl: '',
        objectId: '',
        manifest: '',
        loadedImg: [],
        imgType: 'default',
        extentCoords: [0,0,0,0]

      }
    },

    computed: {
      lmecObjectUrl: function(){ return `https://collections.leventhalmap.org/search/${this.objectId}`; },
      dcObjectUrl: function() { return `https://www.digitalcommonwealth.org/search/${this.objectId}`; },
      arkLink: function() { 
        if(!this.manifest.metadata) { return ''; }
        else {
          var idf = this.manifest.metadata.filter(d => d.label === "Identifier" )[0].value;
          if(typeof idf === 'string' || idf instanceof String){ return idf; } else { return idf[0]; }

        }
 },
      manifestUrl: function() { return `https://www.digitalcommonwealth.org/search/${this.objectId}/manifest.json`; },
      loadedImgIIIF: function() { return this.manifest.sequences && this.loadedImg.length > 0 ? this.manifest.sequences[this.loadedImg[0]].canvases[this.loadedImg[1]].images[this.loadedImg[2]].resource.service["@id"] : ''; },
    },

    methods: {
      resolveObjectUrl: function(e) { 
          const re = /commonwealth:[^//]+/;
          const match = re.exec(this.objectUrl)
          if( match ) {
            this.state = 'loading';
            this.error = false;
            this.objectId = match[0];
            this.getManifest();
          } else {
            this.state = "error";
            this.error = true;
            this.errorMessage = "This URL doesn't seem to be valid. Note: Objects from partner collections, where the original image must be viewed on the partner's site, cannot be used with this tool."
          }

       },
       getManifest: function() {
        //  Edit this pattern for different mappings from collections ID to Manifest
          fetch(this.manifestUrl)
            .then(r => r.json())
            .then(d => {this.manifest = d; this.manifestLoaded = true; })
            .catch(e => {this.state = 'error'; this.error = true; this.errorMessage = 'There is something wrong with the Manifest for this object. Check the URL to make sure it’s valid.'});

       },

       loadImage: function(s,c,i){
          this.loadedImg = [s,c,i];
          this.state='imageLoaded';
          this.loadIIIFLayer(this.loadedImgIIIF + "/info.json");
        },

        loadIIIFLayer(endpoint) {

    fetch(endpoint).then((response) => {
    response.json().then((imageInfo) => {
      var options = new IIIFInfo(imageInfo).getTileSourceOptions();
      if (options === undefined || options.version === undefined) {
        window.alert('Data seems to be no valid IIIF image information.');
        return;
      }
      options.zDirection = -1;
      var iiifTileSource = new IIIF(options);
      this.layer.setSource(iiifTileSource);
      this.map.setView(new View({
        resolutions: iiifTileSource.getTileGrid().getResolutions(),
        extent: iiifTileSource.getTileGrid().getExtent(),
        constrainOnlyCenter: true
      }));
      this.map.getView().fit(iiifTileSource.getTileGrid().getExtent());
      this.map.render();
    }).catch(() => {
      window.alert('Could not read image info json.');
    });
  }).catch(() => {
    window.alert('Could not read data from URL.');
  });
}
    },

  });
</script>

<style>
@import "ol/ol.css";

#ol-map { height: 500px; width: 100%; margin-top: 20px; background-color: #222; }

#main > .container {
	margin-top: 1rem;
}

li > ul {
	margin-left: 15px;
}



</style>
