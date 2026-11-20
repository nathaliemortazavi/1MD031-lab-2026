
<template>
  <div>
    <h3>{{ burger.name }}</h3>

    <img v-bind:src="burger.img" v-bind:alt="burger.name">

    <ul>
      <li>{{ burger.kCal }} kCal</li>
      <li v-if="burger.lactose" class="allergen">
        Contains lactose
      </li>
      <li v-if="burger.gluten" class="allergen">
        Contains gluten
      </li>
    </ul>

    <button type="button" v-on:click="decrease">-</button>
    {{ count }}
    <button type="button" v-on:click="increase">+</button>
  </div>
</template>

<script>
export default {
  name: 'OneBurger',
  props: {
    burger: Object
  },
  data: function () {
    return {
      count: 0
    }
  },
  methods: {
    increase: function () {
      this.count++;
      this.$emit('update-count', this.burger.name, this.count);
    },
    decrease: function () {
      if (this.count > 0) {
        this.count--;
        this.$emit('update-count', this.burger.name, this.count);
      }
    }
  }
}
</script>

<style scoped>
</style>
