<template>
    <div class="slide">
      <slot></slot>
    </div>
</template>

<script>
	export default {
		name: "Slide"
	}
</script>

<style scoped>
  .slide {
    width: 100%;
    flex-shrink: 0;   /* 防止图片被挤压 */
  }

  .slide img {
    width: 100%;
  }
</style>
