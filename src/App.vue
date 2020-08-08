<template>
  <div id="app" class="container">
    <uploader
      :options="{
        baseURL: 'http://image-upload-cropper.test/api',
        maxConcurrentUploads: 1,
      }"
      :handlers="{
        'video/mp4': {
          endpoint: 'upload',
        },
        'image/jpeg': {
          endpoint: 'upload',
        },
      }"
    />
  </div>
</template>

<script>
import Uploader from "./uploader/components/Uploader.vue";
export default {
  name: "App",
  components: {
    Uploader,
  },
};
</script>

<style src="@/assets/styles/app.css" />
