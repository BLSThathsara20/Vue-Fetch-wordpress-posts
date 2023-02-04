<template>
  <div>
    <input type="text" v-model="wpUrl">
    <button @click="validateUrl">Validate URL</button>
    <ul v-for="post in posts" v-bind:key="post.id">
      <li>{{ post.title.rendered }}</li>
    </ul>
    <p v-if="posts.length === 0 && wpUrl" >No posts in this URL</p>
    <span v-if="!wpUrl">Please add URL</span>
    <span v-if="errorMessage">{{ errorMessage }}</span>
    <br>
    <br>
    <h3>History URLs:</h3>
    <ul>
      <li v-for="historyUrl in historyUrls" v-bind:key="historyUrl">
        <a @click="fetchPosts(historyUrl)">{{ historyUrl }}</a>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      wpUrl: "",
      posts: [],
      historyUrls: [],
      errorMessage: ""
    };
  },
  watch: {
    wpUrl: function(newUrl) {
      this.fetchPosts(newUrl);
    }
  },
  methods: {
    fetchPosts(wpUrl) {
      this.wpUrl = wpUrl;
      axios
        .get(wpUrl + "/wp-json/wp/v2/posts")
        .then(response => {
          this.posts = response.data;
          if (!this.historyUrls.includes(wpUrl)) {
            this.historyUrls.push(wpUrl);
          }
          this.errorMessage = "";
        })
        .catch(error => {
          if (error.response.status === 404) {
            this.errorMessage = "Request failed with status code 404";
            this.posts = [];
          }
        });
    },
    validateUrl() {
      let url = this.wpUrl;
      if (!url.startsWith("https://www.")) {
        this.errorMessage = "Invalid URL. Please add 'https://www.' in front of the URL.";
        return;
      }
      if (url.endsWith("/")) {
        url = url.slice(0, -1);
      }
      this.wpUrl = url;
      this.fetchPosts(this.wpUrl);
    }
  },
  mounted() {
    if (localStorage.getItem("historyUrls")) {
      this.historyUrls = JSON.parse(localStorage.getItem("historyUrls"));
      this.fetchPosts(this.historyUrls[this.historyUrls.length - 1]);
    }
  },
  beforeDestroy() {
    localStorage.setItem("historyUrls", JSON.stringify(this.historyUrls));
  }
};
</script>













<!-- Add "scoped" attribute to limit CSS to this component only -->
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
</style>
