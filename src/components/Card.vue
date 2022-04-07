<template>
  <div class="card">
    <slot />
    <v-icon class="icon" @click="iconClick">{{ icons.mdiPencil }}</v-icon>
  </div>
</template>

<script>
import { mdiPencil } from "@mdi/js";
export default {
  name: "Card",
  props: {
    iconClick: Function,
  },
  data: () => ({
    icons: {
      mdiPencil,
    },
  }),
};
</script>

<style scoped>
.card {
  display: flex;
  background: white;
  padding: 1.4rem;
  border-radius: 0.4rem;
  box-shadow: 0 1px 0 rgba(12, 33, 70, 0.25);
  font-size: 1.3rem;
  margin-bottom: 0.8rem;
  cursor: pointer;
  align-items: center;
  justify-content: space-between;
}
.icon {
  position: absolute;
  top: 0;
  right: 0;
}
</style>