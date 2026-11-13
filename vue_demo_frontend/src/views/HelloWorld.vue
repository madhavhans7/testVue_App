<template>
  <div class="hello-container">
    <h1>Hello World!</h1>
    <p>Congratulations, you have logged in successfully!</p>

    <div class="button-container">
      <button @click="openCourseApi" class="btn">Open Course API</button>
      <button @click="fetchCourses" class="btn" :disabled="loading">
        {{ loading ? 'Loading...' : 'Load Courses' }}
      </button>
      <button @click="goBack" class="btn">Return to login page</button>
    </div>

    <!-- Improvement 1: heading only appears after the user clicked Load Courses -->
    <h2 v-if="hasSearched">Course List</h2>

    <!-- Course Table -->
    <table v-if="courses.length > 0" class="course-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Course Code</th>
          <th>Course Title</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="course in courses" :key="course.id">
          <td>{{ course.id }}</td>
          <td>{{ course.code }}</td>
          <td>{{ course.title }}</td>
        </tr>
      </tbody>
    </table>

    <!-- Empty / error state (204 No Content or a failed request) -->
    <div v-else-if="hasSearched">
      <p>{{ statusMessage }}</p>
    </div>

    <!-- Initial State -->
    <div v-else>
      <p>Click the button to load courses.</p>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router';
import { ref } from 'vue';
import http from '../http-common.js';

const router = useRouter();

// Reactive state variables
const courses = ref([]);
const loading = ref(false);
const statusMessage = ref('');
const hasSearched = ref(false); // missing from the lab handout, but the template needs it

// Open the backend API in a new browser tab (raw JSON page)
const openCourseApi = () => {
  window.open(`${http.defaults.baseURL}/courses`, '_blank');
};

// Fetch the same data silently and store it in reactive state
const fetchCourses = async () => {
  loading.value = true;
  statusMessage.value = '';

  try {
    const response = await fetch(`${http.defaults.baseURL}/courses`);

    if (response.status === 204) {
      courses.value = [];
      statusMessage.value = 'No courses found in the database.';
      return;
    }

    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    courses.value = await response.json();
  } catch (error) {
    console.error('Error fetching courses:', error);
    courses.value = [];
    statusMessage.value = 'Failed to load courses. Please check if the backend is running.';
  } finally {
    hasSearched.value = true; // runs on success, 204 and error
    loading.value = false;
  }
};

const goBack = () => {
  localStorage.clear();
  router.push('/');
};
</script>

<style scoped>
.hello-container {
  padding: 40px;
  text-align: center;
}
.button-container {
  display: flex;
  gap: 15px;
  justify-content: center;
  padding: 20px 0;
}
.btn {
  padding: 8px 16px;
  cursor: pointer;
}

/* Improvement 2: nicer table (green header, zebra stripes, hover, rounded corners) */
.course-table {
  width: 100%;
  max-width: 900px;
  margin: 15px auto 0;
  border-collapse: separate;
  border-spacing: 0;
  border: 1px solid #d0d7de;
  border-radius: 8px;
  overflow: hidden;
}
.course-table th,
.course-table td {
  padding: 12px 14px;
  text-align: left;
  border-bottom: 1px solid #e5e9ee;
}
.course-table th {
  background-color: #2d6a4f;
  color: #fff;
  font-weight: 600;
}
.course-table tbody tr:nth-child(even) {
  background-color: #f6f8fa;
}
.course-table tbody tr:hover {
  background-color: #e3f2ea;
}
.course-table tbody tr:last-child td {
  border-bottom: none;
}
</style>
