<template>
  <div id="app">
    <Index />
  </div>
</template>

<script>
import Index from "./components/Index";

export default {
  name: "App",
  components: {
    Index,
  },
};
</script>

<style>
li,
ul,
html,
p,
body {
  margin: 0;
  padding: 0;
  list-style-type: none;
}
html,
body,
#app {
  width: 100%;
  height: 100%;
}
#allmap .anchorBL {
  display: none;
}
</style>
