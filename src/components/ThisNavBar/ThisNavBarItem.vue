<template>
  <div :id="id" class="navbar-item">
    <slot></slot>
  </div>
</template>

<script>
import helper from "../../mixins/helpers";
import common from "../../mixins/common";

export default {
  name: "ThisNavbarItem",
  mixins: [common, helper]
};
</script>
