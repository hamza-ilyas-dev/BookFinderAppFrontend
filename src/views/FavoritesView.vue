<script setup >
import { ref } from "vue";
import { onMounted } from "vue";
import axios from "axios";
import router from "../router";

let favorites = ref([]);
const fetchFavorites = async () => {
  const token = localStorage.getItem("token"); 
  //alert(token);
  if (!token) return;  

  try {
    const response = await axios.get("https://localhost:44331/api/favorites", {
      headers: { 
        Authorization: `Bearer ${token}` ,
      
      },
    });

    // Update store with fetched favorites
    favorites.value = response.data;
  } catch (error) {
    console.error("❌ Error fetching favorites:", error);
  }
};
onMounted(fetchFavorites);

</script>

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl  font-bold mb-4">📚 Favorite Books</h1>
    
    <div v-if="favorites.length > 0" class="books-grid">
      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
        <div v-for="book in favorites" :key="book.book.id" class="bg-white shadow-lg rounded-lg p-4">
          <img :src="book.book.coverImageUrl" alt="Book Cover" class="w-full h-40 object-cover rounded">
          <h2 class="text-lg font-semibold mt-2">{{ book.book.title }}</h2>
          <p class="text-gray-600 mb-2">{{ book.book.author }}</p>
          <div class="flex justify-between items-center">
            <div class="flex items-center">icon</div>
          </div>
        </div>
      </div>
    </div>

    <p v-else>No favorites found.</p>
  </div>
</template>



