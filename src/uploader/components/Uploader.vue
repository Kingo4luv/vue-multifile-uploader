<template>
  <div>
    <uploader-form @chosen="handleFilesChosen" />
    <uploader-file
      :endpoint="determineEndpointFor(upload.file.type)"
      :baseURL="options.baseURL"
      v-for="(upload, i) in uploads"
      :key="i"
      :upload="upload"
    />
  </div>
</template>

<script>
import get from "lodash.get";
import UploaderForm from "./UploaderForm.vue";
import UploaderFile from "./UploaderFile.vue";
import options from "../options";
export default {
  components: {
    UploaderForm,
    UploaderFile,
  },
  props: {
    options: {
      required: false,
      type: Object,
      default: () => options,
    },
    handlers: {
      required: true,
      type: Object,
    },
  },
  data() {
    return {
      uploads: [],
    };
  },
  methods: {
    determineEndpointFor(fileType) {
      return get(this.handlers[fileType], "endpoint", null);
    },
    handleFilesChosen(files) {
      this.uploads.push(
        ...Array.from(files).map((file) => {
          return {
            file,
          };
        })
      );
    },
  },
};
</script>
