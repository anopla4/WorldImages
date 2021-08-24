<template>
  <div class="container">
    <h5>Buscar palabra:</h5>
    <b-form-input
      v-model="word"
      placeholder="Ingrese una palabra"
      id="searchInput"
    ></b-form-input>
    <b-button class="btn" variant="primary" @click="fetchData">Buscar</b-button>
    <p>{{ word }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      word: undefined,
      sellers_id: [1, 2, 3, 4, 5],
      sellers: []
    };
  },
  beforeMount() {
    this.sellers_id.forEach(id => {
      this.$http
        .get(`https://api.alegra.com/api/v1/sellers/${id}`, {
          headers: {
            Authorization:
              "Basic YW5vcGxhNEBnbWFpbC5jb206MWY4NmNjODMxNTc5ZGU0OGRiMWQ="
          }
        })
        .then(response => {
          return response.json();
        })
        .then(data => this.sellers.push(data));
    });
    console.log(this.sellers);
  },

  methods: {
    get_image: function() {},
    fetchData: function() {
      this.$http
        .get("https://serpapi.com/playground?q=Apple&tbm=isch&ijn=0")
        .then(response => {
          const data = response.json();
          console.log(data);
        });
    }
  }
};
</script>

<style>
#searchInput {
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
}
</style>
