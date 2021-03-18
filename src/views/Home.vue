<template>
  <div class="layout">
    <section id="step1" class="section">
      <h1 class="title">Step 1: <span>Upload X-Ray Image</span></h1>
      <vue-dropzone ref="dropzone" id="dropzone" 
        :options="dropzoneOptions" 
        :useCustomSlot="true" 
        v-on:vdropzone-file-added="fileAdded" 
        v-on:vdropzone-sending="sendingRecord" 
        v-on:vdropzone-complete="uploadComplete">
        <div class="dropzone-custom-content">
          <h3 class="dropzone-custom-title">Drag and drop X-Ray image</h3>
          <div class="subtitle">or click to select a file</div>
        </div>
        <div class="dropzone-previews"></div>
      </vue-dropzone>
    </section>
    <section id="step2" class="section">
      <h1 class="title">Step 2: <span>Input patient details</span></h1>
      <input class="input" type="text" name="patient_id" id="patient_id" v-model="details.patient_id" placeholder="Patient ID">
      <input class="input" type="text" name="opacity" id="opacity" v-model="details.opacity" placeholder="Opacity">
      <input class="input" type="text" name="laterality" id="laterality" v-model="details.laterality" placeholder="Laterality">
      <input class="input" type="text" name="lung_zone" id="lung_zone" v-model="details.lung_zone" placeholder="Lung Zone">
      <input class="input" type="text" name="distribution" id="distribution" v-model="details.distribution" placeholder="Distribution">
      <div class="cbx-grp">
        <input type="checkbox" name="normal" id="normal" v-model="details.normal">
        <label for="normal">Normal</label>
      </div>
      <div class="cbx-grp">
        <input type="checkbox" name="pleural_effusion" id="pleural_effusion" v-model="details.pleural_effusion">
        <label for="pleural_effusion">Pleural Effusion</label>
      </div>
      <div class="cbx-grp">
        <input type="checkbox" name="cavitation" id="cavitation" v-model="details.cavitation">
        <label for="cavitation">Cavitation</label>
      </div>
      <input class="input" type="hidden" name="imgurl" id="imgUrl" v-model="details.cxr_image">
      <div class="error" v-html="error"></div>
      <button class="btn" type="submit" v-on:click="submit" v-if="!this.result.normal">Analyze</button>
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
import axios from 'axios'

axios.defaults.withCredentials = true

// @ is an alias to /src
export default {
  name: 'Home',
  components: {
    vueDropzone: vue2Dropzone
  },
  data: ()=> {
    return {
      hasResult: false,
      bucket: 'https://cherish-cxr.s3-ap-southeast-1.amazonaws.com',
      folder: 'images_png',
      findingsEndpoint: 'http://54.255.248.146:8000/findings',
      dropzoneOptions: {
          url: 'https://cherish-cxr.s3-ap-southeast-1.amazonaws.com',
          headers: { 
            'Access-Control-Allow-Origin': '*',
          },
          thumbnailWidth: 150,
          maxFilesize: 100,
          maxFiles: 1,
          acceptedFiles: 'image/png, image/jpeg, */dicom,.dcm, image/dcm, */dcm, .dicom',
          // autoProcessQueue: false,
          addRemoveLinks: true,
          parallelUploads: 1
      },
      details: {
        patient_id: '',
        cxr_image: '',
        normal: false,
        pleural_effusion: false,
        cavitation: false,
        opacity: '',
        laterality: '',
        lung_zone: '',
        distribution: ''
      },
      error: '',
      result: {},
      cookie: ''
    }
  },
  computed: {
    hasResults: function() {
      return !this.result && Object.keys(this.result).length === 0 && this.result.constructor === Object
    }
  },
  methods: {
    login: function() {
      const payload = new URLSearchParams()
      payload.append('email', 'jdgaudillo@gmail.com')
      payload.append('password', 'secret')
      payload.append('role', 'radiologist')

      axios.post('http://54.255.248.146:8000/auth/login', payload)
        .then(function (response) {
          console.log(response)
        })
        .catch(function (error) {
          console.log(error)
        })
      
    },
    submit: function() {
      const dropzone = this.$refs.dropzone.dropzone
      // console.log(this.details, this.findingsEndpoint)
      
      let items = []
      if(this.details.patient_id.length < 1) items.push('<strong>Patient Number</strong>')
      if(dropzone.files.length < 1) items.push('<strong>X-Ray Number</strong>')
      if(this.details.opacity.length < 1) items.push('<strong>Opacity</strong>')
      if(this.details.laterality.length < 1) items.push('<strong>Laterality</strong>')
      if(this.details.lung_zone.length < 1) items.push('<strong>Lung Zone</strong>')
      if(this.details.distribution.length < 1) items.push('<strong>Distribution</strong>')

      this.error = items.length == 0 ? '' : dropzone.files.length < 1 ? 'Please add the X-Ray image.' : items.length == 1 ? `Please dont leave the field ${items[0]} empty.` : `Please fill in the fields ${ (items.toString()).replaceAll(',', ', ') }`

      if(this.error.length == 0) {
        // let cookie = 's%3AWrMxME3OaC_8vuhkAGsXORGTCnn-VLmU.BibRxUp6j%2Bh6VB2tSFdEWUP3gSxCQaNqOGIstAEetbg'
        let config = {
          withCredentials: true,
          // headers: { 
          //     'Cookie': `__utscr=${cookie}`,
          // }
        }
        const payload = new URLSearchParams()
        payload.append('patient_id', this.details.patient_id)
        payload.append('cxr_image', this.details.cxr_image)
        payload.append('normal', this.details.normal)
        payload.append('pleural_effusion', this.details.pleural_effusion)
        payload.append('cavitation', this.details.cavitation)
        payload.append('opacity', this.details.opacity)
        payload.append('laterality', this.details.laterality)
        payload.append('lung_zone', this.details.lung_zone)
        payload.append('distribution', this.details.distribution)

        const t = this
        axios.post(this.findingsEndpoint, payload, config)
        .then(function (response) {
          t.result = response.data.data.items[0]
        })
        .catch(function (error) {
          console.log(error)
        })
      }
    },
    fileAdded: function(file) {
      const dropzone = this.$refs.dropzone.dropzone
      if(dropzone.files.length > 1) {
        dropzone.removeFile(file)
        this.error = 'Please process one file at a time.'
      }else {
        this.error = ''
      }
    },
    sendingRecord: function(file, xhr, formData) {
      // console.log('sending:')
      // console.log(file)
      // console.log(xhr)
      // console.log(formData)
      formData.append('key', `${this.folder}/${file.name}`)
      return xhr
    },
    uploadComplete: function(file) {
      // console.log('complete:', file)
      this.details.cxr_image = `${this.bucket}/${this.folder}/${file.name}`
    }
  },
  mounted: function() {
    // this.login()
  }
};
</script>

<style lang="scss" scoped>
@import "../scss/_vars";

/* eslint-disable */
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

  // &.result {
  //   // grid-column: 1 / span 2;
  // }
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

.cbx-grp {
  display: inline-block;
  margin-right: 0.5rem;
  position: relative;

  &:last-of-type {
    margin-right: 0;
  }

  label {
    display: block;
    position: relative;
    cursor: pointer;
    padding-left: 1.5rem;
    z-index: 1;

    &:before, &:after {
      position: absolute;
    }

    &:before {
      content: '';
      
      display: block;
      width: 1.15rem;
      height: 1.15rem;
      top: 0.1rem;
      left: 0.1rem;
      
      background: #eeeeee;
      background: linear-gradient(to bottom, #fff 0%, #f1f1f1 40%, #eeeeee 100%);
      border-radius: 0.2rem;
      box-shadow: inset 0px 1px 1px white, 0px 1px 2px rgba(0,0,0,0.5);
    }
    
    &:after {
      content: '';
      width: 0.7rem;
      height: 0.3rem;
      position: absolute;
      top: 0.3rem;
      left: 0.3rem;
      border: 3px solid #333;
      border-top: none;
      border-right: none;
      background: transparent;
      opacity: 0;
      transform: rotate(-45deg);
    }
    &:hover::after {
      opacity: 0.5;
    }
  }

  input[type=checkbox] {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 0;
    visibility: hidden;
    &:checked + label:after {
      opacity: 1;
    }
  }
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
  margin-top: 0.5rem;
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
  
  background-color: #eeeeee;
  box-shadow: inset 0px 1px 1px white, 0px 1px 2px rgba(0,0,0,0.1);

  transition: all 0.3s ease-in-out;

  font-size: 0.8rem;
  text-transform: uppercase;
  color: #333;

  cursor: pointer;

  &:hover {
    background-color: $blue;
    color: white;
  }
}

.vue-dropzone {
  height: 75%;
  .dropzone-custom-title {
    font-family: 'Noto Sans JP', sans-serif;
    font-weight: 400;
  }
}
</style>
