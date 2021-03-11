<template>
  <div class="layout">
    <section id="step1" class="section">
      <h1 class="title">Step 1: <span>Input patient details</span></h1>
      <input class="input" type="text" name="patientNo" id="patientNo" v-model="patientNo" placeholder="Patient Number">
      <input class="input" type="text" name="xrayNo" id="xrayNo" v-model="xrayNo" placeholder="X-Ray Number">
      <input class="input" type="text" name="wbc" id="wbc" v-model="wbc" placeholder="White Blood Count">
      <input class="input" type="text" name="temp" id="temp" v-model="temp" placeholder="Temperature">
      <input class="input" type="text" name="lung" id="lung" v-model="lung" placeholder="Lung Findings">
    </section>
    <section id="step2" class="section">
      <h1 class="title">Step 2: <span>Upload X-Ray Image</span></h1>
      <vue-dropzone ref="dropzone" id="dropzone" :options="dropzoneOptions" :useCustomSlot="true" v-on:vdropzone-file-added="fileAdded" v-on:vdropzone-sending="sendingRecord">
        <div class="dropzone-custom-content">
          <h3 class="dropzone-custom-title">Drag and drop X-Ray image</h3>
          <div class="subtitle">or click to select a file</div>
        </div>
        <div class="dropzone-previews"></div>
      </vue-dropzone>
      <div class="error" v-html="error"></div>
      <button class="btn" type="submit" v-on:click="submit">Analyze</button>
    </section>
    
    <!-- <section id="step3" class="section" v-if="hasResult"> -->
    <section id="step3" class="section" v-if="this.result.normal">
      <h1 class="title">Result:</h1>
      <ul class="result-list">
        <li>Normal: <strong>{{ this.result.normal }}</strong></li>
        <li>Covid19: <strong>{{ this.result.covid19 }}</strong></li>
        <li>Viral: <strong>{{ this.result.viral }}</strong></li>
        <li>Bacterial: <strong>{{ this.result.bacterial }}</strong></li>
      </ul>
    </section>
  </div>
</template>

<script>
import vue2Dropzone from 'vue2-dropzone'
import 'vue2-dropzone/dist/vue2Dropzone.min.css'
import axios from "axios"

// @ is an alias to /src
export default {
  name: "Home",
  components: {
    vueDropzone: vue2Dropzone
  },
  data: ()=> {
    return {
      hasResult: false,
      dropzoneOptions: {
          url: 'http://localhost:8080/',
          headers: { "Access-Control-Allow-Origin": "*" },
          thumbnailWidth: 150,
          maxFilesize: 100,
          maxFiles: 1,
          acceptedFiles: 'image/png, image/jpeg',
          autoProcessQueue: false,
          addRemoveLinks: true,
          parallelUploads: 1
      },
      patientNo: '',
      xrayNo: '',
      wbc: '',
      temp: '',
      lung: '',
      error: '',
      result: {}
    }
  },
  computed: {
    hasResults: function() {
      return !this.result && Object.keys(this.result).length === 0 && this.result.constructor === Object
    }
  },
  methods: {
    submit: function() {
      const dropzone = this.$refs.dropzone.dropzone
      
      let items = []
      if(this.patientNo.length < 1) items.push('<strong>Patient Number</strong>')
      if(this.xrayNo.length < 1) items.push('<strong>X-Ray Number</strong>')
      if(this.wbc.length < 1) items.push('<strong>White Blood Count</strong>')
      if(this.temp.length < 1) items.push('<strong>Temperature</strong>')
      if(this.lung.length < 1) items.push('<strong>Lung Findings</strong>')

      this.error = items.length == 0 ? '' : dropzone.files.length < 1 ? 'Please add the X-Ray image.' : items.length == 1 ? `Please dont leave the field ${items[0]} empty.` : `Please fill in the fields ${ (items.toString()).replaceAll(',', ', ') }`

      if(process.env.NODE_ENV == 'development') {
        axios.get('./json/dummy.json')
          .then(response => this.result = response.data)

      } else {
        if(this.error.length == 0) dropzone.processQueue()
      }
    },
    fileAdded: function(file) {
      const dropzone = this.$refs.dropzone.dropzone
      if(dropzone.files.length > 1) {
        dropzone.removeFile(file)
        this.error = 'Please analyze one file at a time.'
      }else {
        this.error = ''
      }
    },
    sendingRecord: function(file, xhr, formData) {
      formData.append('patientNo', this.patientNo)
      formData.append('xrayNo', this.xrayNo)
      formData.append('wbc', this.wbc)
      formData.append('temp', this.temp)
      formData.append('lung', this.lung)
    }
  }
};
</script>

<style lang="scss" scoped>
@import "../scss/_vars";

.layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  column-gap: 2rem;

  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);

  width: 100%;
  padding: 2rem;
  padding-left: $section-pad;
}

.title {
  margin-bottom: 1rem;
}

.section {
  background-color:rgba(255,255,255, 0.8);
  border: 1px solid #f1f1f1;
  padding: 1rem 1.5rem;
  border-radius: .5rem;
  box-shadow: 3px 3px 3px rgba(0,0,0,0.1);
  margin-bottom: 2rem;

  &.result {
    // grid-column: 1 / span 2;
  }
}

.input {
  display: block;
  width: 100%;
  margin: 0 0 1rem;
  padding: 0.5rem;
  // border-radius: 0.3rem;
  border: 0;
  border-bottom: 1px solid #ccc;
  font-size: 1rem;
}

.result-list {
  list-style: none;
  margin: 0;
  padding: 0;
  
  li {
    margin: 0 0 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid #ccc;
  }
}

.error {
  font-size: 0.8rem;
  color: red;
  margin-top: 0.5rem
}

.btn {
  display: block;
  float: right;
  clear: right;

  margin-top: 1rem;
  padding: 0.3rem 0.8rem;
  border: 0;
  background: none;
  border: 1px solid #ccc;
  border-radius: 0.2rem;

  transition: all 0.3s ease-in-out;

  font-size: 0.8rem;
  text-transform: uppercase;
  color: #333;

  cursor: pointer;

  &:hover {
    background-color: #888;
    color: white
  }
}

.vue-dropzone {
  .dropzone-custom-title {
    font-family: 'Noto Sans JP', sans-serif;
    font-weight: 400;
  }
}
</style>
