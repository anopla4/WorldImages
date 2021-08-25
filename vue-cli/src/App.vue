<template>
  <div class="container">
    <b-row class="mb-3" style="text-align:center;">
      <h1>Imágenes del mundo</h1>
    </b-row>
    <b-row>
      <b-col>
        <b-row class="searchInput">
          <b-col
            ><h5>Buscar palabra:</h5>
            <b-form-input
              v-model="word"
              placeholder="Ingrese una palabra"
              id="searchInput"
            ></b-form-input>
            <b-button class="btn" variant="primary" @click="fetchUnsplashData"
              >Buscar</b-button
            ></b-col
          ></b-row
        ><b-row class="mt-3" v-if="bill_created">
          <b-row>
            <b-col
              cols="5"
              v-for="img in images.slice(
                this.first_element,
                this.first_element + 5
              )"
              :key="img.id"
            >
              <b-card
                border-variant="secondary"
                @click="update_points(img.id)"
                v-bind:title="
                  'Vendedor: ' +
                    sellers.find(t => t['id'] === (img.id % 5) + 1)['name']
                "
                v-bind:img-src="img.url"
                img-alt="Image"
                img-top
                tag="article"
                style="max-width: 15rem;"
                class="mb-2 cardImage"
              ></b-card
            ></b-col>
          </b-row>
        </b-row>
        <b-row v-else>
          <b-card class="mt-3 bill" title="Factura"
            ><b-list-group flush>
              <b-list-group-item
                >Nombre del vendedor ganador:
                {{ this.bill["seller"]["name"] }}</b-list-group-item
              >
              <b-list-group-item
                >Producto:
                {{ this.bill["items"][0]["name"] }}</b-list-group-item
              >
              <b-list-group-item
                >Cantidad vendida:
                {{ this.bill["items"][0]["quantity"] }}</b-list-group-item
              >
            </b-list-group></b-card
          ></b-row
        ></b-col
      ><b-col cols="3"
        ><b-card
          class="mt-3"
          bg-variant="light"
          v-for="seller in sellers"
          :key="seller.id"
          :title="seller.name"
          ><b-row
            ><b-col>Puntos acumulados: {{ sellers_points[seller.id] }}</b-col
            ><b-col v-if="sellers_points[seller.id] < 20"
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
      first_element: 0,
      next_images: [],
      sellers_points: { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 }
    };
  },
  beforeMount() {
    this.sellers_id.forEach(id => {
      this.$http
        .get(`https://api.alegra.com/api/v1/sellers/${id}`, {
          headers: {
            Authorization: `Basic YW5vcGxhNEBnbWFpbC5jb206MWY4NmNjODMxNTc5ZGU0OGRiMWQ=`
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
      const n_id = (id % 5) + 1;
      this.sellers_points[n_id.toString()] += 3;
      this.first_element += 5;
      if (this.sellers_points[n_id.toString()] >= 20) {
        this.bill_created = false;
        const sum = Object.values(this.sellers_points).reduce(
          (a, b) => a + b,
          0
        );
        const date = new Date();
        const date_string = `${date.getFullYear()}-${date.getMonth() +
          1}-${date.getDate()}`;
        const bill = {
          date: date_string,
          dueDate: date_string,
          client: 1,
          seller: id,
          items: [
            {
              id: 1,
              price: 3,
              quantity: parseFloat(sum)
            }
          ]
        };
        this.$http
          .post("https://api.alegra.com/api/v1/invoices", bill, {
            headers: {
              Authorization: `Basic YW5vcGxhNEBnbWFpbC5jb206MWY4NmNjODMxNTc5ZGU0OGRiMWQ=`
            }
          })
          .then(response => {
            return response.json();
          })
          .then(data => (this.bill = data));
      }
    },
    fetchUnsplashData: function() {
      for (let index = 1; index < 11; index++) {
        var base_url = `https://api.unsplash.com/search/photos?query=${this.word}?page=${index}`;
        this.$http
          .get(base_url, {
            headers: {
              Authorization: `Client-ID F-W4E8ZVBHBuwHCV87bwAHON8kyMQZlkXsXimEQdTBE`
            }
          })
          .then(response => {
            return response.json();
          })
          .then(
            data =>
              (this.images = this.images.concat(
                data["results"].map((t, index) => {
                  return { id: index + 1, url: t["urls"]["regular"] };
                })
              ))
          );
      }
    }
  }
};
</script>

<style>
.searchInput {
  width: 50%;
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

.bill {
  width: 60%;
}

.cardImage:hover {
  box-shadow: 20px 20px 40px 0px rgba(0, 0, 0, 0.5);
}
</style>
