<script lang="ts">

import axios from 'axios';

interface Joke {
  category: string,
  type: string,
  joke: string,
  flags: {
    nsfw: boolean,
    religious: boolean,
    political: boolean,
    racist: boolean,
    sexist: boolean,
    explicit: boolean
  },
  id: number,
  safe: boolean,
  lang: string
}

export default {
  data() {
    return {
      jokes: [] as Joke[],
      loading: false,
      error: false,
      saving: false,
      favoriteJokes: [] as Joke[],
      selectedCategory: "Any"
    };
  },
  async mounted() {
    try {
      this.loading = true;

      const jokesResponse = await axios.get(`https://v2.jokeapi.dev/joke/${this.selectedCategory}?type=single&amount=10`);
      this.jokes = jokesResponse.data.jokes;

      const favoriteJokesResponse = await axios.get('http://localhost:3004/favoriteJokes');
      this.favoriteJokes = favoriteJokesResponse.data;

      this.loading = false;

    } catch (error) {
      this.error = true;
      this.loading = false;
    }
  },
  methods: {
    async saveJoke(joke: Joke) {
      try {
        this.saving = true

        await axios.post(`http://localhost:3004/favoriteJokes`, joke)
        this.favoriteJokes = [...this.favoriteJokes, joke]

      } catch (error) {
        console.log(error)
      }
      this.saving = false
    },
    isFavorite(joke: Joke): boolean {

      return this.favoriteJokes.some(favoriteJoke => favoriteJoke.id === joke.id)
    },
    async deleteFromFavorites(joke: Joke) {

      const idToRemove = joke.id

      try {
        await axios.delete(`http://localhost:3004/favoriteJokes/${joke.id}`)
        this.favoriteJokes = this.favoriteJokes.filter(joke => joke.id !== idToRemove)

      } catch (error) {
        console.log(error)
      }
    },
    async handleSelectedCategoryChange() {
      try {
        this.loading = true;

        const jokesResponse = await axios.get(`https://v2.jokeapi.dev/joke/${this.selectedCategory}?type=single&amount=10`);
        this.jokes = jokesResponse.data.jokes;

        const favoriteJokesResponse = await axios.get('http://localhost:3004/favoriteJokes');
        this.favoriteJokes = favoriteJokesResponse.data;

        this.loading = false;

      } catch (error) {
        this.error = true;
        this.loading = false;
      }
    }
  },
};

</script>

<template>
  <div class="home">
    <div class="category__container">
      <label class="select__label">Select Joke category</label>
      <select id='category' class="form-select" aria-label="Default select example" v-model="selectedCategory"
        @change="handleSelectedCategoryChange()">
        <option selected value="Any">Any</option>
        <option value="Programming">Programming</option>
        <option value="Dark">Dark</option>
        <option value="Pun">Pun</option>
        <option value="Miscellaneous">Miscellaneous</option>
      </select>
    </div>
    <div v-if="loading" class="loading">Loading...</div>
    <div v-if="error" class="error">error occurred</div>
    <div v-if="jokes" class="jokes__container">
      <div class="joke__Card" v-for="joke in jokes" :key="joke.id">
        <p class="joke__Text">{{ joke.joke }}</p>
        <div class="button__container">
          <button v-if="!isFavorite(joke)" class="button__save" :disabled="saving" @click="saveJoke(joke)">
            save
          </button>
          <button v-if="isFavorite(joke)" class="button__saved" :disabled="saving" @click="deleteFromFavorites(joke)">
            saved
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>

.home {
  display: flex;
  width: 100%;
  flex-direction: column;
  padding: 10vh 0;
  gap: 10px;
}

@media (min-width: 1024px) {
  .home {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    gap: 30px;
    align-items: center;
    justify-content: center;
  }
}

.category__container {
  display: flex;
  width: 100%;
  padding-bottom: 50px;
}

.select__label {
  font-size: 1.25rem;
}

.button__container {
  position: relative;
  transform: rotate(-90deg);
}

.button__save {
  all: unset;
  position: absolute;
  bottom: -27px;
  left: -3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1px 10px 4px 10px;
  border: 1px solid hsla(160, 100%, 37%, 1);
  border-radius: 0 18px;
  font-size: 0.75rem;
  margin: 0;
  transition: background-color 0.25s, color 0.25s;
}

.button__save:hover {
  background-color: hsla(160, 100%, 37%, 1);
  color: black;
}

.button__saved {
  all: unset;
  position: absolute;
  bottom: -27px;
  left: -3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1px 10px 4px 10px;
  border: 1px solid rgb(189, 183, 0);
  border-radius: 0 18px;
  font-size: 0.75rem;
  margin: 0;
  transition: background-color 0.25s, color 0.25s;
}

.button__saved:hover {
  background-color: rgb(189, 183, 0);
  color: black;
}

.jokes__container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.joke__Card {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  padding: 5px 30px;
  width: 100%;
  border: 1px solid transparent;
  border-bottom: 1px solid grey;
  border-radius: 20px;
  transition: background-color 0.25s, border 0.25s;
  min-height: 80px;
}

.joke__Card:hover {
  background-color: rgb(32, 32, 32);
  border: 1px solid hsla(160, 100%, 37%, 1);
}

.joke__Text {
  text-indent: 20px;
  font-size: 1.2rem;
}
</style>