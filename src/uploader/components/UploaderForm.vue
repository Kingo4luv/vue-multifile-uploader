<template>
  <div>
    <form
      enctype="multipart/form-data"
      novalidate
      class="bg-gray-200 border-2 border-gray-400 border-dashed rounded-lg p-10 flex items-center justify-center mb-6 relative"
      :class="{ 'border-gray-600': dragging }"
    >
      <input
        type="file"
        multiple
        class="absolute opacity-0 w-full h-full top-0 left-0"
        @change="handleFilesChosen"
        @dragover.prevent="handleDragOver"
        @dragleave.prevent="handleDragLeave"
      />
      <template v-if="dragging">
        <div class="text-gray-700">
          Nearly there. Let go upload
          <span class="font-medium"> {{ draggingCount }}</span> items!
        </div>
      </template>
      <template v-else>
        <div class="text-gray-700">
          Drop here to upload or <span class="text-blue-500">choose files</span>
        </div>
      </template>
    </form>
  </div>
</template>
<script>
export default {
  data() {
    return {
      dragging: false,
      draggingCount: 0,
    };
  },
  methods: {
    handleFilesChosen(e) {
      this.$emit("chosen", e.target.files);
      this.dragging = false;
    },
    handleDragOver(e) {
      this.dragging = true;
      this.draggingCount = e.dataTransfer.items.length;
    },
    handleDragLeave() {
      this.dragging = false;
      this.draggingCount = 0;
    },
  },
};
</script>
