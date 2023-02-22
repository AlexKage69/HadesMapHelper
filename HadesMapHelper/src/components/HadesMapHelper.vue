<script>
const STATUS_INITIAL = 0,
  STATUS_SAVING = 1,
  STATUS_SUCCESS = 2,
  STATUS_FAILED = 3,
  DIVIDER = 0.35;

import AllJsonImages from "./JsonLibrary.vue";

export default {
  name: "app",
  data() {
    return {
      uploadedfile: null,
      uploadedjson: null,
      obstacle: null,
      HadesDirectory: {},
      AllAssets: {},
      result: null,
    };
  },
  computed: {
    isInitial() {
      return this.currentStatus === STATUS_INITIAL;
    },
    isSaving() {
      return this.currentStatus === STATUS_SAVING;
    },
    isSuccess() {
      return this.currentStatus === STATUS_SUCCESS;
    },
    isFailed() {
      return this.currentStatus === STATUS_FAILED;
    },
  },
  methods: {
    previewFiles(event) {
      this.uploadedfile = event.target.files[0];
      const reader = new FileReader();
      this.result = {};
      reader.onload = (res) => {
        JSON.parse(res.target.result).Obstacles.forEach((value, index) => {
          value.Unused = true;
          this.HadesDirectory[value.Id] = value;
        });
        var count = 0;
        for (const [key, value] of Object.entries(this.HadesDirectory)) {          
            if(this.AllAssets[value.Name] != null){
              this.drawItem(value.Name, value.Location.X, value.Location.Y)
              //console.log(value);
              if(count >= 1){
                break;
              }
              if(this.result[value.Name] == null){
                this.result[value.Name] = 1
              }else{
                this.result[value.Name]++;
              }
              count++;
            }
          };
      };
      reader.onerror = (err) => console.log(err);
      reader.readAsText(this.uploadedfile);
      console.log(this.HadesDirectory)
    },
    drawItem(id, x, y) {
      var self = this
      var info = this.AllAssets[id]
      this.mapContext.scale(-1,-1)  
      var img = new Image();
      img.src = info.src;
      img.onload = function () {
        self.mapContext.drawImage(img, info.rect.x, info.rect.y, info.rect.width, info.rect.height, x*DIVIDER-2000, y*DIVIDER-1000, info.rect.width*DIVIDER, info.rect.height*DIVIDER );
      }
    },
    drawAsset(id, x, y) {
      var self = this
      var info = this.AllAssets[id]
      var img = new Image();
      img.src = info.src;
      img.onload = function () {
        self.assetContext.drawImage(img, info.rect.x, info.rect.y, info.rect.width, info.rect.height, x, y, 180, 180 );
      }
    },
    loadAssets() {
      var self = this;
        Object.keys(AllJsonImages).forEach(function(key) {
          if(key[0] === key[0].toUpperCase() && key[0] != "_"){
            AllJsonImages[key].subAtlases.forEach((value, index)=> {
              value.id = value.name.split('\\').pop().replaceAll("_", "")
              value.originalFile = key
              value.src = "atlases/"+key+".png";
              if(self.AllAssets[value.id] == null || self.AllAssets[value.id].scaleRatio.x > value.scaleRatio.x ){
                self.AllAssets[value.id] = value;
              }
            })
          }
        });
        var height = 0;
        for (const [key, value] of Object.entries(this.AllAssets)) {   
          this.drawAsset(value.id, 0, height)
          height += 180
        }
        if(height < 775){
          this.$refs.canvasAssetRef.height = 775;
        }else{
          this.$refs.canvasAssetRef.height = height
        }
    }
  },
  mounted() {
    this.mapContext = this.$refs.canvasMapRef.getContext('2d');
    this.assetContext =  this.$refs.canvasAssetRef.getContext('2d');
    this.loadAssets();
    this.mapContext.globalCompositeOperation = 'destination-over';
  },
};
//Tartarus_Wall88.atlas.json
</script>

<template>
  <div class="content_holder">
    <div class="canvas_holder">
	    <canvas id="mapCanvas" width="5000" height="5000" ref="canvasMapRef"></canvas>
    </div>
    <div class="assets_list" ref="assetListRef">
	    <canvas width="180" ref="canvasAssetRef"></canvas>
    </div>
    <div class="toolbar" ref="toolbarRef">
      <input
        type="file"
        @change="previewFiles"
        accept=".thing_text, .sjson, .json"
        class="input-file"
    />
    </div>
  </div>
</template>
<style scoped>
</style>