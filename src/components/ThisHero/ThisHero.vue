<template>
  <section :id="id" :class="getClasses">
    <div :class="getHeadClasses">
      <slot name="head"></slot>
    </div>
    <div :class="getBodyClasses">
      <slot></slot>
    </div>
    <div :class="getFootClasses">
      <slot name="foot"></slot>
    </div>
  </section>
</template>

<script>
import syntax from "../../mixins/syntax";
import sizes from "../../mixins/sizes";
import common from "../../mixins/common";
import CssArchitect from "../../utils/css-architect";

export default {
  name: "ThisHero",
  mixins: [common, syntax, sizes],
  props: {
    isBold: {
      type: [String, Boolean]
    },
    headClass: {
      type: String
    },
    bodyClass: {
      type: String
    },
    footClass: {
      type: String
    }
  },
  computed: {
    /**
     * Dynamically build the css classes for the target element
     * @returns { A String with the chained css classes }
     */
    getClasses: function() {
      const cssArchitect = new CssArchitect("hero");
      cssArchitect.addClass(this.getSyntaxModifiers);
      cssArchitect.addClass(this.getSizesModifiers);
      cssArchitect.addClass("is-bold", this.getBoolean(this.isBold));

      return cssArchitect.getClasses();
    },
    /**
     * Dynamically build the css classes for the hero head element
     * @returns { A String with the chained css classes }
     */
    getHeadClasses: function() {
      const cssArchitect = new CssArchitect("hero-head");
      cssArchitect.addClass(this.headClass);
      return cssArchitect.getClasses();
    },
    /**
     * Dynamically build the css classes for the hero body element
     * @returns { A String with the chained css classes }
     */
    getBodyClasses: function() {
      const cssArchitect = new CssArchitect("hero-body");
      cssArchitect.addClass(this.bodyClass);
      return cssArchitect.getClasses();
    },
    /**
     * Dynamically build the css classes for the hero foot element
     * @returns { A String with the chained css classes }
     */
    getFootClasses: function() {
      const cssArchitect = new CssArchitect("hero-foot");
      cssArchitect.addClass(this.footClass);
      return cssArchitect.getClasses();
    }
  },
  data() {
    return {};
  }
};
</script>
