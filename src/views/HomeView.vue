
<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useToast } from "vue-toastification";

const toast = useToast();
const books = ref([]);
const favoriteBooks = ref([]);
const token = localStorage.getItem("token"); 

onMounted(async () => {
  await fetchBooks();
  await fetchFavorites();
});

// Fetch books from API
const fetchBooks = async () => {
  try {
    const response = await axios.get("https://localhost:44331/api/books");
    books.value = response.data;
  } catch (error) {
    console.error("Error fetching books:", error);
  }
};

// Fetch user's favorite books
const fetchFavorites = async () => {
  if (!token) return;
  try {
    const response = await axios.get("https://localhost:44331/api/favorites", {
      headers: { Authorization: `Bearer ${token}` },
    });
    favoriteBooks.value = response.data.map(fav => fav.book.id);
  } catch (error) {
    console.error("Error fetching favorites:", error);
  }
};

// Toggle favorite status of a book
const toggleFavorite = async (bookId) => {
  if (!token) {
    alert("Please log in to favorite books.");
    return;
  }

  const isFavorite = favoriteBooks.value.includes(bookId);
  if (isFavorite) {
    // Remove from favorites
    try {
      const favItem = (await axios.get("https://localhost:44331/api/favorites", {
        headers: { Authorization: `Bearer ${token}` },
      })).data.find(fav => fav.book.id === bookId);

      if (!favItem) return;
      await axios.delete(`https://localhost:44331/api/favorites/${favItem.id}`, {
        headers: { Authorization: `Bearer ${token}` },
      });

      favoriteBooks.value = favoriteBooks.value.filter(id => id !== bookId);
      toast.success("Book remove from favorites.");
    } catch (error) {
      console.error("Error removing favorite:", error);
    }
  } else {
      // Add to favorites
      try { 
      
      const response = await axios.post(
        "https://localhost:44331/api/favorites",
        bookId ,  
        {
          headers: { 
            Authorization: `Bearer ${token}`,
            "Content-Type": "application/json"  
          },
        }
      );

      toast.success("Book added to favorites.");

      // Ensure `response.data` has the expected format before modifying the array
      if (response.data && response.data.bookId) {
        favoriteBooks.value.push(response.data.bookId);
      } else {
        // alert("Unexpected response format");
      }
      router.push("/");
    } catch (error) {
      console.error("Error adding favorite:", error?.response?.data?.message || error.message);
      alert(`Error: ${error?.response?.data?.message || error.message}`);
    }
  }
};
</script>
<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Book List</h1>
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
      <div v-for="book in books" :key="book.id" class="bg-white shadow-lg rounded-lg p-4">
        <img :src="book.coverImageUrl" alt="Book Cover" class="w-full h-40 object-cover rounded">
        <h2 class="text-lg font-semibold mt-2">{{ book.title }}</h2>
        <p class="text-gray-600 mb-2">{{ book.author }}</p>
        <div class="flex justify-between items-center">
          <!-- View Details Button -->
          <RouterLink
            :to="'/books/' + book.id"
            class="mt-5 bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">
            View Details
          </RouterLink>
          
          <!-- Favorite Button with Heart Icon -->
          <button 
  @click="toggleFavorite(book.id)"
  class="mt-2 p-2 rounded flex items-center text-white text-sm"
  :class="favoriteBooks.includes(book.id) ? 'bg-red-500 hover:bg-red-600' : 'bg-blue-500 hover:bg-blue-600'">
  <span class="mr-2">
    <i v-if="favoriteBooks.includes(book.id)" class="fas fa-heart"></i>
    <i v-else class="far fa-heart"></i>
  </span>
  {{ favoriteBooks.includes(book.id) ? 'Remove from Favorites' : 'Add to Favorites' }}
</button>
        </div>
      </div>
    </div>
  </div>
</template>



