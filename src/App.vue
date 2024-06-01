<template>
  <div class="container" style="background-color: dark">
    <h2 style="text-align: center; font-family: Cambria, Cochin, Georgia, Times, 'Times New Roman', serif">My Note</h2>
    <form @submit.prevent="save" class="form">
      <input
        type="text"
        v-model="form.title"
        placeholder="Title"
        class="input"
      />

      <textarea
        v-model="form.content"
        placeholder="Content"
        class="textarea"
      ></textarea>
      <button type="submit" class="button green-button">Save</button>
    </form>

    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <div class="button-group">
          <button @click="edit(article)" class="button green-button">
            Edit
          </button>
          <button @click="deleteArticle(article.id)" class="button red-button">
            Delete
          </button>
        </div>
      </li>
    </ul>

    <button @click="load" class="button green-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    // Reactive form object to hold article data
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    // Ref to hold the list of articles
    const articles = ref([]);

    // Function to load articles from the server
    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles:", error);
      }
    }

    // Function to save or update an article
    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";

        if (!form.id) {
          // Generate a new id as a string in the format "1", "2", "3", etc.
          const newId = (articles.value.length + 1).toString();
          form.id = newId;
        }

        const response = await axios[method](url, form);

        if (method === "put") {
          // Update the article in the list if it already exists
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Add the new article to the list
          articles.value.push(response.data);
        }

        // Reset the form
        form.id = null;
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    // Function to delete an article
    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    }

    // Function to set the form for editing an article
    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    // Load articles when the component is mounted
    onMounted(load);

    // Return the reactive variables and methods to the template
    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background:  #4ec7ff;
  border-radius: 8px;
}

.form {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.input,
.textarea {
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #cccccc;
  border-radius: 4px;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.green-button {
  background: #af4c8b;
  color: #ffffff;
}

.green-button:hover {
  background: #458ea0;
}

.red-button {
  background: #f44336;
  color: #fff;
}

.red-button:hover {
  background: #3547e5;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 4px;
  margin-bottom: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.button-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 10px;
}

.button-group .button {
  margin-left: 10px;
}
</style>
