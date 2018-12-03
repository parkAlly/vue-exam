<template>
  <div class="input-group mb-3 p-2">
    <input type="text" class="input form-control" :value="value" v-on="listeners" >
    <div class="input-group-append">
      <!--<button class="btn btn-outline-secondary" type="button" id="button-addon2" @click="test">조회</button>-->
      <button class="btn btn-outline-secondary" type="button" id="button-addon2" @click="$emit('getinfo')">조회</button>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      value: {
        type: String,
        default: ""
      }
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
