<script setup>
import { ref, watchEffect } from "vue";
import { useRoute } from "vue-router";
import axios from "axios";

const route = useRoute(); // Get route parameters
const selectedBook = ref(null); // Store book details

// Watch for changes in the route and fetch the book data
watchEffect(async () => {
  const bookId = route.params.id;
  if (!bookId) return;

  try {
    const response = await axios.get(`https://localhost:44331/api/books/${bookId}`);
    selectedBook.value = response.data;
  } catch (error) {
    console.error("❌ Error fetching book:", error);
  }
});
</script>

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4"> 📚 Selected Book</h1>
    <div class="flex justify-center items-center">
      <RouterLink to="/" class="text-blue-500 hover:underline"><--  Back to Book List</RouterLink>
    </div>

    <div v-if="selectedBook" class="bg-white shadow-lg rounded-lg p-4 max-w-md mx-auto">
      <img :src="selectedBook.coverImageUrl" alt="Book Cover" class="w-full h-60 object-cover rounded">
      <h2 class="text-lg font-semibold mt-2">{{ selectedBook.title }}</h2>
      <p class="text-gray-600 mb-2">by {{ selectedBook.author }}</p>
      <div class="flex items-center mt-4">icon</div>
    </div>

    <p v-else class="text-gray-500 text-center">Loading book details...</p>
  </div>
</template>
