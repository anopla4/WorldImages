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
        ><b-row v-if="bill_created">
          <b-card-group deck>
            <b-card
              v-for="img in images"
              :key="img.id"
              @click="update_points(img.id.toString())"
              v-bind:title="
                'Vendedor: ' + sellers.find(t => t['id'] === img.id)['name']
              "
              v-bind:img-src="img.url"
              img-alt="Image"
              img-top
              tag="article"
              style="max-width: 15rem;"
              class="mb-2"
            ></b-card>
          </b-card-group>
        </b-row>
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
      images: [],
      first_element: 1,
      next_images: [],
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
      if (this.next_images.length !== 0) {
        this.images = [...next_images];
        this.next_images = [];
      } else {
        this.first_element += 10;
        this.fetchData();
      }
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
      gapi.client.setApiKey("AIzaSyDFanaXHAyTj9KPXAlH5VpBRqEUB7PZTQk");
      return gapi.client
        .load(
          "https://content.googleapis.com/discovery/v1/apis/customsearch/v1/rest"
        )
        .then(
          () => {
            console.log("GAPI client loaded for API");
          },
          err => {
            console.error("Error loading GAPI client for API", err);
          }
        )
        .then(() => {
          return gapi.client.search.cse
            .list({
              cx: "6ce5fdff9abf5811f",
              q: this.word,
              searchType: "image",
              start: this.first_element
            })
            .then(
              function(response) {
                return response["result"]["items"];
              },
              function(err) {
                console.error("Execute error", err);
              }
            );
        })
        .then(img => {
          this.images = img.slice(0, 5).map((item, index) => {
            return { id: index + 1, url: item["link"] };
          });
          this.next_images = img.slice(5, 10).map((item, index) => {
            return { id: index + 1, url: item["link"] };
          });
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
