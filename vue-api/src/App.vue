<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (method === 'post') {
          articles.value.push(response.data);
        } else {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  },
};
</script>

<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Masukkan Judul" class="input"/><br />
      <textarea v-model="form.content" placeholder="Masukkan Pesan" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>

    <ul class="articles">
      <li v-for="article in articles" :key="article.id" class="article">
        <strong>{{ article.title }}</strong><br />
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="remove(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>

    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<style>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Courier New', Courier, monospace;
}

.form {
  margin-bottom: 20px;
  padding: 100px 200px;
  width: 100%;
  border: 1px solid rgb(255, 255, 255);
  border-radius: 8px;
  background-color: #f193cd;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.button {
  padding: 10px 20px;
  margin: 10px 0;
  border: none;
  border-radius: 4px;
  background-color: #5a6af8;
  color: rgb(255, 255, 255);
  cursor: pointer;
}

.button:hover {
  background-color: #92bce9;
}

.articles {
  list-style-type: none;
  padding: 0;
}

.article {
  padding: 20px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #f680cd;
}

.edit-button {
  background-color: #52d3f7;
}

.edit-button:hover {
  background-color: #3bd0f5;
}

.delete-button {
  background-color: #dc3545;
}

.delete-button:hover {
  background-color: #c82333;
}

.load-button {
  background-color : rgb(73, 128, 248);;
}

.load-button:hover {
  background-color: #5d95ff;
}
</style>
