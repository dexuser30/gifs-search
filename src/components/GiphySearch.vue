<template>
   <div class="hello">
      <h1>Giphy search</h1>
      <p>
         Search query term or phrase
      </p>

      <p>
         <input
             v-model="query"
             type="text"
             class="giphy-search__input"
             @input="search"
         >
      </p>

      <ShowGif
          v-if="selectedGif"
          :gif="selectedGif"
      />

      <div class="giphy__container">
         <Loader v-if="loading"/>

         <div
             v-if="!loading"
             v-for="gif in gifs"
             class="giphy__item"
             :key="gif.id"
         >
            <img
                :src="gif.images.preview_gif.url"
                :alt="gif.title"
                @click="showGif(gif)"
            >
         </div>

         <Loader v-if="loadingMore"/>
      </div>
   </div>
</template>

<script>
import axios from 'axios'
import Loader from "./Loader"
import {debounce} from "debounce"
import ShowGif from "./ShowGif"

export default {
   name: 'GiphySearch',
   components: {
      ShowGif,
      Loader
   },
   data: () => ({
      query: null,
      searchDelay: 300,
      offset: 0,
      totalCount: null,
      loading: true,
      loadingMore: false,
      gifs: [],
      selectedGif: null,
      trendingGifs: [],
      apiUrl: `https://api.giphy.com/v1/gifs/`,
      apiKey: '?api_key=dc6zaTOxFJmzC',
      limit: 45,
      error: null,
   }),
   methods: {
      search(type) {
         if (!this.query) {
            this.gifs = this.trendingGifs
            this.loading = false

            return
         }

         if(type === 'load_more') {
            this.loadingMore = true
            this.offset += this.limit
         }
         else {
            this.loading = true
            this.offset = 0
         }

         axios
             .get(`${this.apiUrl}search${this.apiKey}&q=${this.query}&offset=${this.offset}&limit=${this.limit}`)
             .then((result) => {
                if(this.offset == 0) {
                   this.gifs = result.data.data
                   this.totalCount = result.data.pagination.total_count
                   this.loading = false
                }
                else {
                   this.gifs = this.gifs.concat(result.data.data)
                   this.loadingMore = false
                }
             })
             .catch(error => {
                console.log(error);
             })
      },
      getTrending() {
         axios
             .get(`${this.apiUrl}trending${this.apiKey}&limit=${this.limit}`)
             .then((result) => {
                this.trendingGifs = [...result.data.data]
                this.gifs = this.trendingGifs
             })
             .then((result) => {
                this.loading = false
             })
             .catch(error => {
                console.log(error);
             })
      },
      showGif(gif) {
         this.selectedGif = gif
         this.selectedGif.show = true
      }
   },
   created() {
      // Get trending gifs
      this.getTrending()
      // Search with delay
      this.search = debounce(this.search, this.searchDelay)
      // Search load more
      window.addEventListener('scroll', () => {
         if(this.offset > this.totalCount || this.loadingMore) return
         if((window.innerHeight + window.scrollY) >= document.body.offsetHeight) this.search('load_more')
      })
   }
}
</script>

<style scoped lang="scss">
h3 {
   margin: 40px 0 0;
}

ul {
   list-style-type: none;
   padding: 0;
}

li {
   display: inline-block;
   margin: 0 10px;
}

a {
   color: #42b983;
}

.giphy-search {
   &__input {
      width: 100%;
      max-width: 500px;
      height: 45px;
      margin: 0;
      padding: 0 15px;
      border: 1px solid #42b983;
   }
}

.giphy {
   &__container {
      display: flex;
      flex-wrap: wrap;
      width: 100%;
      max-width: 1000px;
      margin: 0 auto;
      padding: 50px;
   }

   &__item {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 190px;
      height: 190px;
      margin: 5px;
      cursor: pointer;

      img {
         width: 100%;
         height: 100%;
         object-fit: cover;
      }
   }
}
</style>
