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
      favoriteJokes: [] as Joke[],
      loading: false,
      error: false,
      saving: false,
    };
  },
  async mounted() {
    try {
      this.loading = true;
      const favoriteJokesResponse = await axios.get('http://localhost:3004/favoriteJokes');
      this.favoriteJokes = favoriteJokesResponse.data;
      console.log(favoriteJokesResponse)
      this.loading = false;
    } catch (error) {
      this.error = true;
      this.loading = false;
    }
  },
  methods: {
    async deleteFromFavorites(joke: Joke) {
      const idToRemove = joke.id
      try {
        await axios.delete(`http://localhost:3004/favoriteJokes/${joke.id}`)
        this.favoriteJokes = this.favoriteJokes.filter(joke => joke.id !== idToRemove)
      } catch (error) {
        console.log(error)
      }
    },
  },
};

</script>

<template>
  <div class="favoriteJokes">
    <div v-if="loading" class="loading">Loading...</div>
    <div v-if="favoriteJokes.length === 0" class="loading">Not saved any jokes yet, go to homepage to save some!</div>
    <div v-if="error" class="error">error occurred</div>
    <div v-if="favoriteJokes" class="jokes__container">
      <div class="joke__Card" v-for="joke in favoriteJokes" :key="joke.id">
        <p class="joke__Text">{{ joke.joke }}</p>
        <div class="deleteButton__container">
          <button class="deleteButton__button" @click="deleteFromFavorites(joke)">
            delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.favoriteJokes {
  display: flex;
  flex-direction: column;
  padding: 10vh 0;
  gap: 10px;
}

@media (min-width: 1024px) {
  .favoriteJokes {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    gap: 30px;
    align-items: center;
    justify-content: center;
  }
}

.deleteButton__container {
  position: relative;
  transform: rotate(-90deg);
}

.deleteButton__button {
  all: unset;
  position: absolute;
  bottom: -27px;
  left: -3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1px 10px 4px 10px;
  border: 1px solid rgb(189, 0, 28);
  border-radius: 0 18px;
  font-size: 0.75rem;
  margin: 0;
  transition: background-color 0.25s, color 0.25s;
}

.deleteButton__button:hover {
  background-color: rgb(189, 0, 28);
  color: rgb(255, 255, 255);
}

.jokes__container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.joke__Card {
  display: flex;
  position: relative;
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
