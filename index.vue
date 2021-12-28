<template>
  <label :style="'height:'+height+';width:'+width+';line-height:'+height+';text-align:center;color:white;'">
    <button v-if="label">[[label]]</paper-button>
    <v-icon v-if="!label" style="margin-top:-3px;margin-left:3px;">{{icon}}</v-icon>
    <img id="image" :src="src" :width="width" :height="height" :style="'max-width:100px;max-height:70px;'+trans" />
    <input style="position: fixed;top: -1000px" accept="image/*" type="file" id="flies" on-change="handleFileSelect" name="file" />
  </label>
</template>
<script>
import exif from "exif-js"

export default {
  name: "load-image",
    props:{
      icon: {
        default: "mdi-camera-add",
        type:String,
      },
      label:{
        default: "",
        type:String,
      },
      src:{
        default: "",
        notify:true,
      },
      orientation:{
        default: 0,
        notify:true,
      },
      exif:{
        notify:true,
      },
    },
    computed:{
      trans: function(){
        return this.getTrans(this.orientation)
      },
    },
    methods: {
      getTrans: function(o){
        if (+o > 0) {
          var s = ["","transform: scale(-1, 1)","transform: rotate(180deg)", "transform: rotate(180deg)  scale(-1, 1)", "transform: rotate(90deg)  scale(1, -1)","transform: rotate(90deg)", "transform: rotate(-90deg)  scale(1, -1)", "transform: rotate(-90deg)"]
          return s[o-1]
        } else {
          return o
        }
      },
      handleFileSelect: function(evt){
        var that = this  
        var file = evt.target.files; // FileList object
      
        // Loop through the FileList and render image files as thumbnails.
        for (var i = 0, f; f = file[i]; i++) {
      
          // Only process image files.
          if (!f.type.match('image.*')) {
            continue;
          }
      
          var reader = new FileReader();
      
          // Closure to capture the file information.
          reader.onload = (function(theFile) {
            return function(e) {
              // output value
              that.src = e.target.result
            }
          })(f)
       
          // Read in the image file as a data URL.
          reader.readAsDataURL(f)
          this.getOrientation(f, function(orientation) {
            that.orientation = orientation
          })
          this.getExif(f)
        }
      },
      getExif: function(file) {
        var that = this
        var reader3 = new FileReader()
        reader3.onload = function(e) {
          if (EXIF) {
            that.exif = EXIF.readFromBinaryFile(this.result)
          }
        }
        reader3.readAsArrayBuffer(file)
      },
      getOrientation: function(file, callback) {
        var reader2 = new FileReader()
        reader2.onload = function(e) {
        var view = new DataView(e.target.result)
        if (view.getUint16(0, false) != 0xFFD8) return callback(-2);
        var length = view.byteLength, offset = 2;
        while (offset < length) {
          var marker = view.getUint16(offset, false);
          offset += 2;
          if (marker == 0xFFE1) {
            if (view.getUint32(offset += 2, false) != 0x45786966) return callback(-1);
            var little = view.getUint16(offset += 6, false) == 0x4949;
            offset += view.getUint32(offset + 4, little);
            var tags = view.getUint16(offset, little);
            offset += 2;
            for (var i = 0; i < tags; i++)
              if (view.getUint16(offset + (i * 12), little) == 0x0112)
                return callback(view.getUint16(offset + (i * 12) + 8, little))
              } else if ((marker & 0xFF00) != 0xFF00) break
            else offset += view.getUint16(offset, false)
          }
          return callback(-1)
        }
        reader2.readAsArrayBuffer(file.slice(0, 64 * 1024))
      },
    }
  }
</script>
<style>
  :host {
    background: grey;
    min-height: 30px;
    min-width: 30px;
    display: inline-block;
  }
  label { 
    min-height: 50px;
    min-width: 50px;
    display: inline-block;
  }
</style>
