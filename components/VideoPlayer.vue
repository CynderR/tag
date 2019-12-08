<template>
  <b-col cols="12">
    <video id="videoPlayer" @dragend.prevent="dragend" controls :src="fileToBlob(file)"></video>
  </b-col>
</template>
<script>
export default {
  name: 'VideoPlayer',
  props: ['file'],
  methods: {
    dragend(e) {
      console.log(e);
    },
    fileToBlob() {
      if (!this.file) {
        return;
      }
      return URL.createObjectURL(this.file);
    }
  }
}
</script>
