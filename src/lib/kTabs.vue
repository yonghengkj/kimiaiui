<template>
  <div class="k-tabs">
    <div class="k-tabs-nav" ref="container">
      <div
        :class="{ selected: t === selected }"
        class="k-tabs-nav-item"
        v-for="(t, index) in titles"
        :ref="
          (el) => {
            if (t === selected) selectedItem = el;
          }
        "
        :key="index"
        @click="select(t)"
      >
        {{ t }}
      </div>
      <div class="k-tabs-nav-indicator" ref="indicator"></div>
    </div>
    <div class="k-tabs-content">
      <component
        class="k-tabs-content-item"
        v-for="(c, index) in defaults"
        :key="index"
        :is="c"
        :class="{ selected: c.props.title === selected }"
      ></component>
    </div>
  </div>
</template>

<script lang='ts'>
import { onMounted, ref, watchEffect } from "vue";
import kTab from "./kTab.vue";
export default {
  props: {
    selected: {
      type: String,
    },
  },
  setup(props, context) {
    const selectedItem = ref<HTMLDivElement>(null);
    const indicator = ref<HTMLDivElement>(null);
    const container = ref<HTMLDivElement>(null);
    onMounted(() => {
      watchEffect(() => {
        const { width } = selectedItem.value.getBoundingClientRect();
        indicator.value.style.width = width + "px";
        const { left: left1 } = container.value.getBoundingClientRect();
        const { left: left2 } = selectedItem.value.getBoundingClientRect();
        const left = left2 - left1;
        indicator.value.style.left = left + "px";
      });
    });
    const defaults = context.slots.default();
    console.log(defaults);
    
    defaults.forEach((tag) => {
      if (tag.type.name !== kTab.name) {
        throw new Error("kTabs组件的子组件必须是kTab"); //防御性编程
      }
    });
    const titles = defaults.map((tag) => {
      return tag.props.title;
    });
    const select = (title: String) => {
      context.emit("update:selected", title);
    };
    return { defaults, titles, select, selectedItem, indicator, container };
  },
};
</script>

<style lang='scss'>
$blue: #40a9ff;
$color: #333;
$border-color: #d9d9d9;
.k-tabs {
  &-nav {
    display: flex;
    color: $color;
    border-bottom: 1px solid $border-color;
    position: relative;
    &-item {
      padding: 8px 0;
      margin: 0 16px;
      cursor: pointer;
      &:first-child {
        margin-left: 0;
      }
      &.selected {
        color: $blue;
      }
    }
    &-indicator {
      height: 3px;
      background-color: $blue;
      position: absolute;
      left: 0;
      bottom: -1px;
      width: 100px;
      transition: all 250ms;
    }
  }
  &-content {
    padding: 8px 0;
    &-item {
      display: none;
    }
    .selected {
      display: block;
    }
  }
}
</style>