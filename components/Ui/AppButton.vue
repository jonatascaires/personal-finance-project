<template>
  <button
    :class="btnColor == 'indigo' ? class1 : class2"
    @click="$emit('click')"
  >
    <slot />
  </button>
</template>

<script>
export default {
  name: "AppButton",
  props: {
    btnColor: {
      type: String,
      default: "indigo",
    },
  },
  data() {
    return {
      class1:
        "inline-flex items-center justify-center border focus:outline-none transition ease-in-out duration-150 text-white bg-indigo-600 hover:bg-indigo-800 border-indigo-600 hover:border-indigo-800 rounded text-sm px-4 py-2",
      class2:
        "inline-flex items-center justify-center border focus:outline-none transition ease-in-out duration-150 text-white bg-red-600 hover:bg-red-800 border-red-600 hover:border-red-800 rounded text-sm px-4 py-2",
    };
  },

  methods: {},
};
</script>
