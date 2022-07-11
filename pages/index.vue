<template>
  <div class="home">
    <Hero ref="mychild" />

    <div class="container search">
      <input
        @keyup.enter="$fetch"
        type="text"
        placeholder="Keywords"
        v-model.lazy="searchInput"
      />
      <button
        @click="clearSearch(), changeF('Now Streaming'), $fetch()"
        v-show="searchInput !== ''"
        class="button"
      >
        Clear Search
      </button>
      <div class="nav">
        <button
          @click="changeU('Trending Past Week'), clearSearch(), $fetch()"
          class="button"
        >
          Trending
        </button>
        <button
          @click="changeT('What\'s Popular'), clearSearch(), $fetch()"
          class="button"
        >
          Popular
        </button>
        <button
          @click="changeF('Now Streaming'), clearSearch(), $fetch()"
          class="button"
        >
          Streaming
        </button>
      </div>
    </div>
    <Loading v-if="$fetchState.pending" />
    <div v-else class="container movies">
      <div v-if="searchInput !== ''" id="movie-grid" class="movies-grid">
        <div
          class="movie"
          v-for="(movie, index) in searchedMovies"
          :key="index"
        >
          <div class="movie-img">
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title }}
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>
            <NuxtLink
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
              >Get More Info</NuxtLink
            >
          </div>
        </div>
      </div>
      <div
        v-if="searchInput === '' && flag === ''"
        id="movie-grid"
        class="movies-grid"
      >
        <div class="movie" v-for="(movie, index) in movies" :key="index">
          <div class="movie-img">
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title }}
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>
            <NuxtLink
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
              >Get More Info</NuxtLink
            >
          </div>
        </div>
      </div>
      <div
        v-if="searchInput === '' && flag === '1'"
        id="movie-grid"
        class="movies-grid"
      >
        <div class="movie" v-for="(movie, index) in popMovies" :key="index">
          <div class="movie-img">
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title }}
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>
            <NuxtLink
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
              >Get More Info</NuxtLink
            >
          </div>
        </div>
      </div>
      <div
        v-if="searchInput === '' && flag === '2'"
        id="movie-grid"
        class="movies-grid"
      >
        <div class="movie" v-for="(movie, index) in upComing" :key="index">
          <div class="movie-img">
            <img
              :src="`https://image.tmdb.org/t/p/w500/${movie.poster_path}`"
            />
            <p class="review">{{ movie.vote_average }}</p>
            <p class="overview">{{ movie.overview }}</p>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title }}
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString('en-us', {
                  month: 'long',
                  day: 'numeric',
                  year: 'numeric',
                })
              }}
            </p>
            <NuxtLink
              class="button button-light"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
              >Get More Info</NuxtLink
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Hero from '../components/Hero.vue'
import Loading from '../components/Loading.vue'
export default {
  name: 'home',
  head() {
    return {
      title: this.appTitle,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: 'Get all the latest streaming movies',
        },
        {
          hid: 'keywords',
          name: 'keywords',
          content: 'latest, streaming, movies',
        },
      ],
    }
  },
  data() {
    return {
      movies: [],
      searchedMovies: [],
      popMovies: [],
      upComing: [],
      searchInput: '',
      flag: '',
      appTitle: '',
    }
  },
  async fetch() {
    console.log(this.flag)
    if (this.searchInput === '' && this.flag === '') {
      await this.getMovies()
      return
    }
    if (this.searchInput === '' && this.flag === '1') {
      await this.popularMovies()
      return
    }
    if (this.searchInput === '' && this.flag === '2') {
      await this.upMovies()
      return
    } else {
      this.changeF(this.searchInput)
      await this.searchMovies()
      return
    }
  },
  fetchOnServer: false,
  fetchDelay: 1000,

  methods: {
    async getMovies() {
      this.appTitle = 'Latest Streaming Movies'
      this.flag = ''
      const data = axios.get(
        `https://api.themoviedb.org/3/movie/now_playing?api_key=460781cf13d89c9998933001675ff5d0&page=1`
      )
      const result = await data
      this.movies = []

      result.data.results.forEach((movie) => {
        this.movies.push(movie)
      })
      console.log(this.movies)
    },
    async popularMovies() {
      this.appTitle = 'Popular Movies'
      const data = axios.get(
        `https://api.themoviedb.org/3/movie/popular?api_key=460781cf13d89c9998933001675ff5d0&page=1`
      )
      const result = await data
      this.popMovies = []

      result.data.results.forEach((movie) => {
        this.popMovies.push(movie)
      })
      console.log(this.popMovies)
    },
    async upMovies() {
      this.appTitle = 'Weekly Trending Movies'
      const data = axios.get(
        `https://api.themoviedb.org/3/trending/all/week?api_key=460781cf13d89c9998933001675ff5d0&page=1`
      )
      const result = await data
      this.upComing = []

      result.data.results.forEach((movie) => {
        this.upComing.push(movie)
      })
      console.log(this.upComing)
    },
    async searchMovies() {
      this.appTitle = 'Gallery'
      this.flag = ''
      const data = axios.get(
        `https://api.themoviedb.org/3/search/movie?api_key=460781cf13d89c9998933001675ff5d0&language=en-US&page=1&query=${this.searchInput}`
      )
      const result = await data
      this.searchedMovies = []
      result.data.results.forEach((movie) => {
        this.searchedMovies.push(movie)
      })
      console.log(this.searchedMovies)
    },
    clearSearch() {
      this.searchInput = ''
      this.searchedMovies = []
    },
    changeT(title) {
      this.flag = '1'
      this.$refs.mychild.changeTitle(title)
    },
    changeF(title) {
      this.flag = ''
      this.$refs.mychild.changeTitle(title)
    },
    changeU(title) {
      this.flag = '2'
      this.$refs.mychild.changeTitle(title)
    },
  },
  components: { Loading, Hero },
}
</script>

<style lang="scss" scoped>
.home {
  .loading {
    padding-top: 120px;
    align-items: flex-start;
  }

  .search {
    display: flex;
    padding: 32px 16px;

    input {
      max-width: 350px;
      width: 100%;
      padding: 12px 6px;
      font-size: 14px;

      &:focus {
        outline: none;
      }
    }

    .button {
      border-top-left-radius: 0;
      border-bottom-left-radius: 0;
    }
  }
  .nav {
    display: flex;
    margin-left: auto;
  }

  .movies {
    padding: 32px 16px;
    .movies-grid {
      display: grid;
      column-gap: 32px;
      row-gap: 64px;
      grid-template-columns: 1fr;
      @media (min-width: 500px) {
        grid-template-columns: repeat(2, 1fr);
      }
      @media (min-width: 750px) {
        grid-template-columns: repeat(2, 1fr);
      }
      @media (min-width: 1100px) {
        grid-template-columns: repeat(4, 1fr);
      }

      .movie {
        position: relative;
        display: flex;
        flex-direction: column;

        .movie-img {
          position: relative;
          overflow: hidden;

          &:hover {
            .overview {
              transform: translateY(0);
            }
          }

          img {
            display: block;
            width: 100%;
            height: 100%;
          }

          .review {
            position: absolute;
            top: 0;
            left: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            width: 40px;
            height: 40px;
            background-color: #c92502;
            color: #fff;
            border-radius: 0 0 16px 0;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1),
              0 2px 4px -1px rgba(0, 0, 0, 0.06);
          }

          .overview {
            line-height: 1.5;
            position: absolute;
            bottom: 0;
            background-color: rgba(201, 38, 2, 0.75);
            padding: 12px;
            color: #fff;
            transform: translateY(100%);
            transition: 0.2s ease-in-out all;
          }
        }

        .info {
          margin-top: auto;
          .title {
            margin-top: 8px;
            color: #fff;
            font-size: 20px;
          }

          .release {
            margin-top: 8px;
            color: #c9c9c9;
          }

          .button {
            margin-top: 8px;
          }
        }
      }
    }
  }
}
</style>
