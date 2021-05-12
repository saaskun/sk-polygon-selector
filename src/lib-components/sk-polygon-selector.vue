<script>
import { computed, ref, onMounted, onBeforeUnmount, watch } from "vue";

export default {
  props: {
    image: {
      type: String,
      required: true
    },
    modelValue: {
      type: Array,
      required: false,
      default: [
        { x: 10, y: 10 },
        { x: 90, y: 10 },
        { x: 90, y: 90 },
        { x: 10, y: 90 }
      ]
    },
    options: {
      type: Object,
      default() {
        return {};
      }
    }
  },
  setup(props, { emit }) {
    const area = ref(null);

    const areaWidth = ref(100);
    const areaHeight = ref(100);
    const intOptions = ref({
      pointSize: 2.5,
      lineStyle: "stroke:rgb(255,0,0)",
      pointStyle: "fill:rgb(255,0,0,0.5)",
      pointDraggedStyle: "fill:rgb(0,255,0,0.5)"
    });
    const points = ref([]);
    const pointOnDrag = ref(null);

    const lines = computed(() => {
      let pointsCount = points.value.length;
      return points.value.map((_point, _index, _points) => {
        return {
          x1: _point.x,
          y1: _point.y,
          x2: pointsCount == _index + 1 ? _points[0].x : _points[_index + 1].x,
          y2: pointsCount == _index + 1 ? _points[0].y : _points[_index + 1].y
        };
      });
    });

    const dragViewArea = computed(() => {
      return "0 0 " + areaWidth.value + " " + areaHeight.value;
    });

    const normalizePoint = evt => {
      return {
        x: (evt.offsetX / areaWidth.value) * 100,
        y: (evt.offsetY / areaHeight.value) * 100
      }; 
    }

    const moveSelection = evt => {
      if (pointOnDrag.value != null) {
        const normalizedPoint = normalizePoint(evt)
        points.value[pointOnDrag.value] = normalizedPoint;
      }
    }

    const selectPoint = evt => {
      const normalizedPoint = normalizePoint(evt)
      let collisionPoints = points.value.filter(
        point =>
          PointUtil.distance(point, normalizedPoint) <= intOptions.value.pointSize
      );

      if (collisionPoints.length > 0) {
        pointOnDrag.value = points.value.indexOf(collisionPoints[0])
      }
    };

    const stopSelection = evt => {
      pointOnDrag.value = null
      changed();
    }

    const onImageLoad = () => {
      onResize();
    };

    const onResize = () => {
      if (area.value) {
        areaWidth.value = area.value.width;
        areaHeight.value = area.value.height;
      }
    };

    const changed = () => {
      emit("update:modelValue", Object.assign([],points.value));
    };

    const startDrag = point => {
      isDragging.value = point;
    };

    const endDrag = point => {
      isDragging.value = point;
    };

    onMounted(() => {
      intOptions.value = Object.assign(intOptions.value, props.options);
      if (props.modelValue) {
        points.value = Object.assign([],props.modelValue);
      }
      window.addEventListener("resize", onResize);
    });

    onBeforeUnmount(() => {
      window.removeEventListener("resize", onResize);
    });

    watch(() => area.height, onResize);

    return {
      intOptions,
      area,
      areaWidth,
      areaHeight,
      dragViewArea,
      lines,
      points,
      pointOnDrag,
      moveSelection,

      changed,
      endDrag,
      onImageLoad,
      onResize,
      selectPoint,
      startDrag,
      stopSelection
    };
  }
};

class PointUtil {
  static distance(p1, p2) {
    return Math.sqrt((p1.x - p2.x) ** 2 + (p1.y - p2.y) ** 2);
  }
}
</script>

<template>
  <div class="selection-area">
    <img
      :src="image"
      alt="Sample"
      ref="area"
      @load="onImageLoad"
    />
    <div class="point-selection">
      <svg :viewBox="dragViewArea" @mousedown="selectPoint" @mousemove="moveSelection" @mouseup="stopSelection">
        <line
          v-for="(line, pIndex) of lines"
          :key="pIndex"
          :x1="line.x1 + '%'"
          :y1="line.y1 + '%'"
          :x2="line.x2 + '%'"
          :y2="line.y2 + '%'"
          :style="intOptions.lineStyle"
        />
        <circle
          v-for="(point, pIndex) of points"
          :key="pIndex"
          :cx="point.x + '%'"
          :cy="point.y + '%'"
          :r="intOptions.pointSize + '%'"
          :style="pointOnDrag==pIndex?intOptions.pointDraggedStyle:intOptions.pointStyle"
        />
      </svg>
    </div>
  </div>
</template>

<style scoped>
.selection-area {
  display: block;
  position: relative;
}

.selection-area img {
  width: 100%;
}

.selection-area .point-selection {
  top: 0px;
  left: 0px;
  right: 0px;
  bottom: 0px;
  position: absolute;
}

.selection-area .point-selection svg {
  width: 100%;
  height: 100%;
}
</style>
