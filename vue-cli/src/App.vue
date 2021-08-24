<template>
  <div class="container">
    <b-row>
      <b-col class="searchInput">
        <b-row>
          <b-col
            ><h5>Buscar palabra:</h5>
            <b-form-input
              v-model="word"
              placeholder="Ingrese una palabra"
              id="searchInput"
            ></b-form-input>
            <b-button class="btn" variant="primary" @click="fetchData"
              >Buscar</b-button
            ></b-col
          ></b-row
        ><b-row v-if="bill_created"
          ><div
            v-for="seller in sellers"
            :key="seller.id"
            class="image"
            @click="update_points(seller.id)"
          ></div
        ></b-row>
        <b-row v-else
          ><p>Nombre del vendedor ganador: {{ this.bill["seller"]["name"] }}</p>
          <p>Producto: {{ this.bill["items"][0]["name"] }}</p>
          <p>Cantidad vendida: {{ this.bill["items"][0]["price"] }}</p></b-row
        ></b-col
      ><b-col
        ><b-card
          class="mt-3"
          bg-variant="light"
          v-for="seller in sellers"
          :key="seller.id"
          :title="seller.name"
          ><b-row
            ><b-col>Puntos acumulados: {{ sellers_points[seller.id] }}</b-col
            ><b-col
              >Puntos faltantes: {{ 20 - sellers_points[seller.id] }}</b-col
            ></b-row
          >
        </b-card></b-col
      ></b-row
    >
  </div>
</template>

<script>
export default {
  data() {
    return {
      word: undefined,
      bill_created: true,
      bill: undefined,
      sellers_id: [1, 2, 3, 4, 5],
      sellers: [],
      sellers_points: { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 }
    };
  },
  beforeMount() {
    this.sellers_id.forEach(id => {
      this.$http
        .get(`https://api.alegra.com/api/v1/sellers/${id}`, {
          headers: {
            Authorization: `Basic ${process.env.VUE_APP_CREDENTIALS}`
          }
        })
        .then(response => {
          return response.json();
        })
        .then(data => this.sellers.push(data));
    });
  },

  methods: {
    update_points: function(id) {
      this.sellers_points[id] += 3;
      if (this.sellers_points[id] >= 20) {
        this.bill_created = false;
        const sum = this.sellers.reduce((a, b) => a + b, 0);
        const date = new Date();
        const date_string = `${date.getFullYear()}-${date.getMonth() +
          1}-${date.getDate()}`;
        const bill = {
          date: date_string,
          dueDate: date_string,
          client: process.env.VUE_APP_CLIENT,
          seller: id,
          items: [
            {
              id: process.env.VUE_APP_PRODUCT,
              price: 3,
              quantity: parseFloat(sum)
            }
          ]
        };
        this.$http
          .post("https://api.alegra.com/api/v1/invoices", bill, {
            headers: {
              Authorization: `Basic ${process.env.VUE_APP_CREDENTIALS}`
            }
          })
          .then(response => {
            return response.json();
          })
          .then(data => (this.bill = data));
      }
    },
    fetchData: function() {
      this.$http
        .get("https://serpapi.com/playground?q=Apple&tbm=isch&ijn=0")
        .then(response => {
          return response.json();
        });
    }
  }
};
</script>

<style>
searchInput {
  width: 50%;
  display: flex;
  justify-content: center;
}

.header {
  width: 100%;
}

.container {
  margin: 5%;
}

.btn {
  float: right;
  margin-top: 3%;
}

.image {
  background-color: blue;
  width: 50px;
  height: 50px;
  margin: 2%;
}
</style>
