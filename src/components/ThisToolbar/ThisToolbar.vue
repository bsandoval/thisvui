<template>
  <div :id="id" :class="getClasses">
    <slot></slot>
  </div>
</template>
<script>
import colors from "../../mixins/colors";
import common from "../../mixins/common";
import flex from "../../mixins/flex-direction";
import alignment from "../../mixins/alignment";
import CssArchitect from "../../utils/css-architect";

export default {
  name: "ThisToolbar",
  mixins: [common, colors, flex, alignment],
  props: {
    isVertical: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    /**
     * Dynamically build the css classes for the target element
     * @returns { A String with the chained css classes }
     */
    getClasses: function() {
      const cssArchitect = new CssArchitect("this-toolbar is-flex");
      cssArchitect.addClass(this.getColorsModifiers);
      cssArchitect.addClass(this.getFlexModifiers);
      cssArchitect.addClass(this.getAlignmentModifiers);
      cssArchitect.addClass("is-vertical", this.getBoolean(this.isVertical));
      return cssArchitect.getClasses();
    }
  },
  data: function() {
    return {};
  }
};
</script>
