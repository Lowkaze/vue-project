<script>
export default {
  data() {
    return {
      posts: [],
      comments: [],
      selectedPost: {},
      search: ''
    }
  },
  computed: {
    postsWithComments() {
      return this.posts.map((post) => {
        const postComments = this.comments.filter((comment) => comment.postId === post.id)

        post.comments = postComments

        return post
      })
    },
    commentsModal() {
      return new bootstrap.Modal(this.$refs.commentsModal)
    },
    filteredPosts() {
      const search = this.search.trim().toLowerCase()

      if (!search) {
        return this.postsWithComments
      }

      return this.postsWithComments.filter((post) => {
        return (
          post.title.trim().toLowerCase().includes(search) ||
          post.body.trim().toLowerCase().includes(search) ||
          `Post ${post.id}`.trim().toLowerCase().includes(search)
        )
      })
    }
  },
  mounted() {
    this.fetchPosts()
    this.fetchComments()
  },
  methods: {
    async fetchPosts() {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts')
      const posts = await response.text()

      this.posts = JSON.parse(posts.replace(/\\n/g, ' '))
    },
    async fetchComments() {
      const response = await fetch('https://jsonplaceholder.typicode.com/comments')
      const comments = await response.json()

      this.comments = comments
    },
    openCommentsModal(postId) {
      this.selectedPost = this.postsWithComments.find((post) => post.id === postId)

      this.commentsModal.show()
    }
  }
}
</script>

<template>
  <div class="container py-5">
    <input type="text" class="form-control mb-4" placeholder="Search" v-model="search" />
    <div class="row g-4">
      <div class="col-12 col-md-6 col-lg-4 col-xl-3" v-for="post in filteredPosts" :key="post.id">
        <div class="card h-100">
          <div class="card-header">Post {{ post.id }}</div>
          <div class="card-body">
            <h5 class="card-title">{{ post.title }}</h5>
            <p class="card-text">{{ post.body }}</p>
          </div>
          <div class="card-footer text-body-secondary">
            <button class="btn btn-secondary" @click="openCommentsModal(post.id)">Comments</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="modal fade" ref="commentsModal" tabindex="-1">
    <div class="modal-dialog modal-dialog-centered modal-dialog-scrollable">
      <div class="modal-content">
        <div class="modal-header">
          <h1 class="modal-title fs-5">Comments for post {{ selectedPost.id }}</h1>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <ul class="list-group list-group-flush">
            <li class="list-group-item" v-for="comment in selectedPost.comments" :key="comment.id">
              {{ comment.body }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>
