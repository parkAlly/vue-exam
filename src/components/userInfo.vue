<template>
  <div class="row p-2">
    <div class="col-3">{{title}}</div>
    <div class="col-9">{{value}}</div>
  </div>
</template>

<script>
  export default {
    props: {
      title: {
        type: String,
        default: ""
      },
      value: {
        type: String,
        default: ""
      },
    },
    computed: {
      listeners () {
        return {
          // Pass all component listeners directly to input
          ...this.$listeners,
          // Override input listener to work with v-model
          input: event => this.$emit('input', event.target.value)
        }
      }
    }
  }
</script>
