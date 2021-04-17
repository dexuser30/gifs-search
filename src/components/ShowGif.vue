<template>
   <div
       class="giphy__selected-gif-container"
       @click="close"
   >
      <img
          :src="gif.images.original.url"
          :alt="gif.title"
      >
   </div>
</template>

<script>
export default {
   name: "ShowGif",
   props: ['gif'],
   methods: {
      close() {
         this.$parent.selectedGif = null
      }
   }
}
</script>

<style scoped lang="scss">
.giphy {
   &__selected-gif-container {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(240, 246, 252, 0.7);
      backdrop-filter: blur(4px);
      display: flex;
      justify-content: center;
      align-items: center;

      img {
         max-width: 640px;
         max-height: 480px;
      }
   }
}
</style>