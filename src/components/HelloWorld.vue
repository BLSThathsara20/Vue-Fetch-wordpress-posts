<template>
  <div>
    <span v-if="posts" class="warning">! Please add only wordpress site url</span>
    <input type="text" v-model="wpUrl">
    <div class="errors">
      <p v-if="posts.length === 0 && wpUrl" >No posts in this URL</p>
    <span class="add-url" v-if="!wpUrl">Please add URL</span>
    <span v-if="errorMessage">{{ errorMessage }}</span>
    </div>
    <p class="endpoint para" v-if="wpUrl"><span>EndPoint: </span> <span class="endpoint">{{wpUrl + "/wp-json/wp/v2/posts"}}</span></p>
    <button class="validate" @click="validateUrl" v-if="wpUrl">Validate URL</button>
    
    <div v-if="!wpUrl" class="example">
      <span class="example-url" @click="updateInputValue">Add Exmple URL</span>
    </div>
    <div class="history">
      <h3>History URLs:</h3>
    <ul>
      <li class="histry-url" v-for="historyUrl in historyUrls" v-bind:key="historyUrl">
        <a @click="fetchPosts(historyUrl)">{{ historyUrl }}</a>
      </li>
    </ul>
    <span class="previous-urls" v-if="wpUrl">You can click these previous urls</span>
    </div>
    <h4 v-if="posts">Post Titles</h4>
    <ul class="posts" v-for="post in posts" v-bind:key="post.id">
      <li><a href="{{ post.link }}">{{ post.title.rendered }}</a> <span>{{ post.content.rendered }}</span></li>
    </ul>
    <br>
    <br>
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
      errorMessage: "",
    };
  },
  watch: {
    wpUrl: function(newUrl) {
      this.fetchPosts(newUrl);
    }
  },
  methods: {
    updateInputValue() {
      this.wpUrl = "https://slpi.lk";
    },
    fetchPosts(wpUrl) {
    if (wpUrl.endsWith("/")) {
      wpUrl = wpUrl.slice(0, -1);
    }
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
      }
      this.posts = []; // clear the posts data
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

.errors{
  padding-top: 10px;
  p,span{
    font-size: 13px;
    color: #ff0000;

    &.add-url{
      color: #0048ff;
      }
  }
}

.posts{
  text-align: left;
  li{
    a{
      font-size: 16px;
      color: #373737;
    }
    span{
      font-size: 12px;
      color: #00000081;
      display: block;
      padding-top: 8px;
      padding-bottom: 12px;

    }
  }
}
.previous-urls{
  font-size: 12px;
  background-color: #00b7ff2d;
  border-radius: 18px;
  padding: 4px 15px;
  color: #003df4
}
.validate{
  padding: 8px 25px;
  border: none;
  background-color: #ff6a00;
  border-radius: 18px;
  color: #fff;
  cursor: pointer;
  display: block;
  max-width: none;
  margin: 10px auto;
}
.bind-url{
  font-size: 14px;
  text-decoration: dashed;
}
.warning{
  display: block;
  margin-bottom: 15px;
  font-size: 12px;
  color: #ff0000;
  background-color: #ff7b002d;
  border-radius: 18px;
  padding: 4px 15px;
}
.example{
  padding-top: 15px;
}
.example-url{
  background-color: #23a84b;
  border-radius: 18px;
  color: #fff;
  padding: 6px 30px;
  cursor: pointer;
}
input{
  padding: 4px 15px;
  font-size: 16px;
  width: 70%;
  border-radius: 4px;
  border: 2px solid #279e68;
}
.history{
  border: 1px solid #9a9a9a;
  padding: 8px;
  margin: 15px 0;

  h3{
    margin-top: 0;
  }
}
.endpoint{
  text-decoration: underline;
  font-size: 14px;
  color: #0046b6;

  &.para{
    color: #212121;
  }
}
.histry-url{
  cursor: pointer;
  display:block;
}
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
