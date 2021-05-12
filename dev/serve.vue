<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "ServeDev",
  data() {
    return {
      points: [
        { x: 20, y: 10 },
        { x: 80, y: 10 },
        { x: 90, y: 90 },
        { x: 10, y: 90 },
        { x: 10, y: 50 }
      ],
      image:  require('./samples/card-example.png')
    };
  }
});
</script>

<template>
  <div id="app">
    <sk-polygon-selector
      :image="image"
      style="width:70%; margin:0px auto;"
      v-model="points"
    />
    {{ points }}
  </div>
</template>
