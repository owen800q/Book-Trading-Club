<template>
  <div>
    <h1>Books</h1>
    <p>Your book collection.</p>
    <router-link class="btn btn-primary" to="/books-add">Add Books to your collection</router-link>
    <hr/>
    <div class="progress progress-striped active" v-if="loading">
      <div class="progress-bar" style="width: 100%"></div>
    </div>
    <div v-else>
      <template v-if="books && books.length > 0">
        <div class="row">
          <div class="col-md-2 col-sm-3 col-xs-6" v-for="book in books" :key="book._id">
            <div class="book">
              <div class="controls">
                <i class="fa fa-trash text-danger" aria-hidden="true" @click="removeBook(book)" title="Remove"></i>
              </div>
              <img alt="thumbnail" :title="book.title" :src="book.thumbnail" />
            </div>
          </div>
        </div>
      </template>
      <template v-else>
        <div class="alert alert-info">
          <h3>Oops!</h3>
          <p>No books found!</p>
        </div>
        <router-link to="/books-add">Click here</router-link> to add books.
      </template>
    </div>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  import axios from 'axios'

  export default {
    data () {
      return {
        books: null,
        loading: true
      }
    },
    computed: {
      ...mapGetters(['getAPI'])
    },
    created () { },
    methods: {
      removeBook (book) {
        if (!confirm("Delete this book form your collection?")) return

        this.loading = true
        axios.delete(this.getAPI.url + '/api/books/' + book._id).then(() => {
          this.books = this.books.filter((item) => {
            return item._id !== book._id
          })
        }).catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
      },
      getBooks () {
        axios.get(this.getAPI.url + '/api/books/self/').then(res => {
          this.books = res.data
        }).catch(err => {
          console.log(err)
        }).then(() => {
          this.loading = false
        })
      }
    },
    mounted () {
      this.getBooks()
    }
  }
</script>

<style>

</style>
