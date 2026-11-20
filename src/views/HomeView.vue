
<template>
  <div>
    <header>
      <div class="hero">
        <h1>Welcome to Natta's Pub</h1>
        <img src="/img/burger_background.png" alt="Restaurant background">
      </div>
    </header>

    <main>
      <section id="burgers">
        <h2>Select burger</h2>
        <p>This is where you execute burger selection.</p>

        <div class="burger-grid">
          <Burger
            v-for="burger in burgers"
            v-bind:burger="burger"
            v-bind:key="burger.name"
            v-on:update-count="updateCount"
          />
        </div>
      </section>

      <section id="customer">
        <h2>Customer information</h2>
        <p>This is where you provide necessary information.</p>

        <h3>Delivery information:</h3>

        <form id="customerForm" v-on:submit.prevent="addOrder">
          <p>
            <label for="fullname">Full name</label><br>
            <input type="text" id="fullname" name="fullname"
                   v-model="customer.name"
                   placeholder="First and Last name" required>
          </p>

          <p>
            <label for="email">E-mail</label><br>
            <input type="email" id="email" name="email"
                   v-model="customer.email"
                   placeholder="E-mail address" required>
          </p>

          <p>
            <label for="payment">Payment options</label><br>
            <select id="payment" name="payment"
                    v-model="customer.payment">
              <option value="credit-card">Credit card</option>
              <option value="debit-card">Debit card</option>
              <option value="paypal">PayPal</option>
              <option value="swish">Swish</option>
            </select>
          </p>

          <p>
            Gender<br>

            <input type="radio" id="male" name="gender"
                   value="male" v-model="customer.gender">
            <label for="male">Male</label><br>

            <input type="radio" id="female" name="gender"
                   value="female" v-model="customer.gender">
            <label for="female">Female</label><br>

            <input type="radio" id="undisclosed" name="gender"
                   value="undisclosed" v-model="customer.gender">
            <label for="undisclosed">Do not wish to provide</label>
          </p>
        </form>
      </section>

      <h3>Choose delivery location:</h3>

      <div class="map-window">
        <div id="map" v-on:click="setLocation">
          <div
            v-if="selectedLocation"
            class="location-marker"
            v-bind:style="{
              left: selectedLocation.x + 'px',
              top: selectedLocation.y + 'px'
            }"
          >
            T
          </div>
        </div>
      </div>

      <button type="submit" form="customerForm">
        🍔 Place my order!
      </button>

      <section id="receipt" v-if="receipt">
        <h2>Order receipt</h2>

        <p><strong>Order #{{ receipt.orderId }}</strong></p>

        <h3>Your burgers:</h3>
        <ul>
          <li
            v-for="(count, burgerName) in receipt.orderItems"
            v-bind:key="burgerName"
          >
            {{ burgerName }}: {{ count }}
          </li>
        </ul>

        <h3>Customer information:</h3>
        <p>
          Name: {{ receipt.customer.name }}<br>
          E-mail: {{ receipt.customer.email }}<br>
          Payment: {{ receipt.customer.payment }}<br>
          Gender: {{ receipt.customer.gender }}
        </p>

        <p>Delivery location selected on map.</p>
      </section>
    </main>

    <footer>
      <hr>
      <p>&copy; 2018 Hypothetical Burgers Inc.</p>
    </footer>
  </div>
</template>

<script>
import Burger from '../components/OneBurger.vue'
import io from 'socket.io-client'
import menu from '../assets/menu.json'

function MenuItem(name, kCal, img, lactose, gluten) {
  this.name = name;
  this.kCal = kCal;
  this.img = img;
  this.lactose = lactose;
  this.gluten = gluten;
}

const socket = io("localhost:3000");

export default {
  name: 'HomeView',

  components: {
    Burger
  },

  data: function () {
    return {
      burgers: menu,
      orderCounts: {},
      selectedLocation: null,
      receipt: null,
      customer: {
        name: '',
        email: '',
        payment: 'credit-card',
        gender: 'undisclosed'
      }
    }
  },

  methods: {
    updateCount: function (burgerName, count) {
      this.orderCounts[burgerName] = count;
    },

    getOrderNumber: function () {
      return Math.floor(Math.random() * 100000);
    },

    setLocation: function (event) {
      const rect = event.currentTarget.getBoundingClientRect();

      this.selectedLocation = {
        x: Math.round(event.clientX - rect.left),
        y: Math.round(event.clientY - rect.top)
      };
    },

    addOrder: function () {
      if (!this.selectedLocation) {
        alert("Please select a delivery location on the map.");
        return;
      }

      const selectedBurgers = {};

      for (const burger of this.burgers) {
        const count = this.orderCounts[burger.name] || 0;

        if (count > 0) {
          selectedBurgers[burger.name] = count;
        }
      }

      if (Object.keys(selectedBurgers).length === 0) {
        alert("Please select at least one burger.");
        return;
      }

      const newOrder = {
        orderId: this.getOrderNumber(),
        details: { ...this.selectedLocation },
        orderItems: { ...selectedBurgers },
        customer: { ...this.customer }
      };

      socket.emit("addOrder", newOrder);

      this.receipt = newOrder;
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=EB+Garamond:wght@400;700&display=swap');


body {
    font-family: 'EB Garamond', Georgia, serif;
    font-size: 18px;
    background-color: #FFF4C2;
}

.allergen {
    font-weight: bold;
}


#burgers {
    background-color: #C99A32;
    color: black;
}


main section {
    margin: 20px 0;
    padding: 15px;
    border: 2px dashed gray;
}

.burger-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15px;
}

.burger-grid > div {
    padding: 10px;
}

button {
    margin-top: 15px;
    margin-bottom: 15px;
}

button:hover {
    background-color: orange;
    cursor: pointer;
}

.hero {
    position: relative;
    height: 220px;
    overflow: hidden;
}

.hero img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    opacity: 0.5;
}

.hero h1 {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 1;
    width: 90%;
    text-align: center;
    margin: 0;
}

.burger-grid img {
    width: 150px;
    height: 220px;
    object-fit: cover;
}

.map-window {
    width: 600px;
    height: 400px;
    overflow: scroll;
}

#map {
    position: relative;
    width: 1920px;
    height: 1078px;
    background-image: url('/img/polacks.jpg');
    cursor: crosshair;
}

.location-marker {
    position: absolute;
    transform: translate(-50%, -50%);
    font-weight: bold;
    color: black;
    pointer-events: none;
}
</style>
