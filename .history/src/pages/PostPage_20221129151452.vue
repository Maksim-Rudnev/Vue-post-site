<template>
  <div>
    <div class="header">
      <h1>Страница с постами</h1>
      <my-select
        v-model="selectedPagination"
        :options="paginationOptions">
      </my-select>
    </div>
    <my-input
      v-model="searchQuery"
      placeholder="Поиск..."
    />
    <div class="app__btns">
      <my-button
        @click="showDialog">
        Создать пост
      </my-button>
      <my-select
        v-model="selectedSort"
        :options="sortOptions"
      />
    </div>

    <my-dialog v-model:show="dialogVisibleError">
      <h1 style="color: brown"> {{this.error}} </h1>
    </my-dialog>

    <my-dialog v-model:show="dialogVisible">
      <post-form @create="createPost" />
    </my-dialog>

    <post-list
      :posts="sortedAndSearchedPosts"
      @remove="removePost"
      v-if="!isPostLoading"
    />
    <div v-else>Идет загрузка...</div>
    <div class="page__wrapper">
      <div
        v-for="pageNumber in totalPage"
        :key="pageNumber"
        class="page"
        :class="{
          'current-page' : page === pageNumber
        }"
        @click="changePage(pageNumber)">
        {{ pageNumber }}
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import PostList from '@/components/PostList.vue';
import PostForm from '@/components/PostForm.vue';

export default {
  components: {
    PostList,
    PostForm,
  },
  data() {
    return {
      posts: [],
      page: 1,
      limit: 8,
      totalPage: 0,
      dialogVisible: false,
      isPostLoading: false,
      error: null,
      dialogVisibleError: false,
      searchQuery: '',
      selectedSort: '',
      sortOptions: [
        { value: 'title', name: 'По названию' },
        { value: 'body', name: 'По описанию' },
      ],
      selectedPagination: 'byPage',
      paginationOptions: [
        { value: 'byPage', name: 'По страницам' },
        { value: 'lenta', name: 'Лента' },
      ],
    };
  },
  methods: {
    createPost(post) {
      this.posts.push(post);
      this.dialogVisible = false;
    },
    removePost(post) {
      this.posts = this.posts.filter((el) => el.id !== post.id);
    },
    showDialog() {
      this.dialogVisible = true;
    },
    changePage(pageNumber) {
      this.page = pageNumber;
    },
    async fetchPost() {
      try {
        this.isPostLoading = true;
        const responce = await axios.get('https://jsonplaceholder.typicode.com/posts', {
          params: {
            _page: this.page,
            _limit: this.limit,
          },
        });
        this.totalPage = Math.ceil(responce.headers['x-total-count'] / this.limit);
        this.posts = responce.data;
      } catch (err) {
        this.error = err;
        this.dialogVisibleError = true;
      } finally {
        this.isPostLoading = false;
      }
    },
  },
  mounted() {
    this.fetchPost();
  },
  watch: {
    page() {
      this.fetchPost();
    },
  },
  computed: {
    sortedPost() {
      return [...this.posts].sort((post1, post2) => post1[this.selectedSort]
        ?.localeCompare(post2[this.selectedSort]));
    },
    sortedAndSearchedPosts() {
      return this.sortedPost.filter((post) => post.title.toLowerCase()
        .includes(this.searchQuery.toLowerCase()));
    },
  },
};
</script>

<style>

.app__btns {
  display: flex;
  justify-content: space-between;
  margin: 15px 0;
}
.page__wrapper {
  display: flex;
  margin-top: 15px;
}
.page {
  border: 1px solid black;
  padding: 10px;
}
.current-page {
  border: 2px solid teal;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
