<template>
  <div class="container">
    <h5>Buscar palabra:</h5>
    <b-form-input
      v-model="word"
      placeholder="Ingrese una palabra"
      id="searchInput"
    ></b-form-input>
    <button @click="fetchData">Buscar</button>
    <p>{{ word }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return { word: undefined };
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
</style>
