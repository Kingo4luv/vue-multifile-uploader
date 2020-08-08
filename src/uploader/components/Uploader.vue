<template>
  <div>
    <uploader-form @chosen="handleFilesChosen" />
    <div class="mb-4 flex justify-between px-4 text-gray-600 text-sm">
      <span
        >{{ this.uploads.length }} uploads (currentUploadCount in progress /
        {{ completedUploadCount }} completed)</span
      >
      <span>{{ overProgress }}% complete</span>
    </div>
    <uploader-file
      :endpoint="determineEndpointFor(upload.file.type)"
      :baseURL="options.baseURL"
      v-for="(upload, i) in uploads"
      :key="i"
      :upload="upload"
      @progress="handleUploadProgress"
      @change="handleUploadChange"
    />
  </div>
</template>

<script>
import { v4 as uuid } from "uuid";
import states from "../state";
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
  computed: {
    currentUploadCount() {
      return this.uploads.filter((upload) => upload.uploading).length;
    },
    completedUploadCount() {
      return this.uploads.filter((upload) => upload.completed).length;
    },
    overProgress() {
      if (this.uploads.length === 0) {
        return 0;
      }
      let uploads = this.uploads.filter(
        (upload) => !upload.cancelled && !upload.failed
      );
      if (this.uploads.length === 0) {
        return 0;
      }
      return parseInt(
        uploads.reduce((a, b) => a + b.progress, 0) / uploads.length
      );
    },
  },
  methods: {
    determineEndpointFor(fileType) {
      return get(this.handlers[fileType], "endpoint", null);
    },
    handleUploadChange(e) {
      switch (e.state) {
        case states.UPLOADING:
          this.uploads = this.uploads.map((upload) => {
            if (e.id === upload.id) {
              upload.uploading = true;
            }
            return upload;
          });
          break;
        case states.COMPLETED:
          this.uploads = this.uploads.map((upload) => {
            if (e.id === upload.id) {
              upload.completed = true;
              upload.uploading = false;
            }
            return upload;
          });
          break;
        case states.CANCELLED:
          this.uploads = this.uploads.map((upload) => {
            if (e.id === upload.id) {
              upload.uploading = false;
              upload.completed = false;
              upload.cancelled = true;
            }
            return upload;
          });
          break;
        case states.FAILED:
          this.uploads = this.uploads.map((upload) => {
            if (e.id === upload.id) {
              upload.uploading = false;
              upload.completed = false;
              upload.cancelled = false;
              upload.failed = true;
            }
            return upload;
          });
          break;
      }
    },
    handleUploadProgress(e) {
      this.uploads = this.uploads.map((upload) => {
        if (e.id === upload.id) {
          upload.progress = e.progress;
        }
        return upload;
      });
    },
    handleFilesChosen(files) {
      this.uploads.push(
        ...Array.from(files).map((file) => {
          return {
            id: uuid(),
            progress: 0,
            uploading: false,
            completed: false,
            cancelled: false,
            failed: false,
            queued: true,
            file,
          };
        })
      );
    },
  },
  mounted() {
    setInterval(() => {
      if (this.currentUploadCount >= this.options.maxConcurrentUploads) {
        return;
      }
      let queued = this.uploads.filter((upload) => upload.queued === true);
      if (queued.length) {
        queued[0].queued = false;
      }
    }, 500);
  },
};
</script>
