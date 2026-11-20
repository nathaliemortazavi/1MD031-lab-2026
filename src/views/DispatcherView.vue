
<template>
  <div id="orders">
    <div id="orderList">
      <div v-for="(order, key) in orders" v-bind:key="'order' + key">

        <strong>#{{ key }}</strong>

        <div v-for="(count, burgerName) in order.orderItems"
             v-bind:key="burgerName">
          {{ burgerName }}: {{ count }}
        </div>

        <div v-if="order.customer">
          Name: {{ order.customer.name }}<br>
          E-mail: {{ order.customer.email }}<br>
          Payment: {{ order.customer.payment }}<br>
          Gender: {{ order.customer.gender }}
        </div>

        <p>Status: {{ order.status || 'New' }}</p>

        <button
          type="button"
          v-on:click="changeStatus(key, 'in preparation')"
        >
          In preparation
        </button>

        <button
          type="button"
          v-on:click="changeStatus(key, 'done')"
        >
          Done
        </button>

        <hr>
      </div>

      <button v-on:click="clearQueue">Clear Queue</button>
    </div>

    <div id="dots">
      <div
        v-for="(order, key) in orders"
        v-bind:style="{
          left: order.details.x + 'px',
          top: order.details.y + 'px'
        }"
        v-bind:key="'dots' + key"
      >
        {{ key }}
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client'

const socket = io("localhost:3000");

export default {
  name: 'DispatcherView',

  data: function () {
    return {
      orders: null
    }
  },

  created: function () {
    socket.on('currentQueue', data => this.orders = data.orders);
  },

  methods: {
    clearQueue: function () {
      socket.emit('clearQueue');
    },

    changeStatus: function (orderId, status) {
      socket.emit('changeStatus', {
        orderId: orderId,
        status: status
      });
    }
  }
}
</script>

<style>
#orderList {
    top: 1em;
    left: 1em;
    position: absolute;
    z-index: 2;
    color: black;
    background: rgba(255, 255, 255, 0.5);
    padding: 1em;
}

#dots {
    position: relative;
    margin: 0;
    padding: 0;
    background-repeat: no-repeat;
    width: 1920px;
    height: 1078px;
    cursor: crosshair;
    background-image: url('/img/polacks.jpg');
}

#dots div {
    position: absolute;
    background: black;
    color: white;
    border-radius: 10px;
    width: 20px;
    height: 20px;
    text-align: center;
}
</style>
