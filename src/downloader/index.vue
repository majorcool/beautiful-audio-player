<template>
    <a :download="target"
       class="link"
       :style="style||{}"
       href="">
        <slot :style="{cursor: 'pointer'}"></slot>
    </a>
</template>

<script>
export default {
  name: "downloader",
  props: {
    target: String,
    style: Object
  }
};
</script>

<style scoped>
.link {
  text-decoration: none;
}
.link * {
  cursor: pointer;
}
</style>


