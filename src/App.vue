<template>
  <div id='app'>
    <router-view />
  </div>
</template>

<script>
export default {
  name: "App"
};

function adjustFontSize() {
  const html = document.querySelector("html");
  let fontSize = window.innerWidth / 10;
  fontSize = fontSize > 50 ? 50 : fontSize;
  html.style.fontSize = fontSize + "px";
}

// 加载时根据窗口大小修改字体尺寸
document.addEventListener("DOMContentLoaded", () => {
  adjustFontSize();
});
//屏幕变化时修改字体尺寸
window.onresize = function() {
  const html = document.querySelector("html");
  adjustFontSize();
};
</script>

<style>
</style>
