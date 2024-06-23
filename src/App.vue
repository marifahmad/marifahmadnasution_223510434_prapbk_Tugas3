<template>
  <section>
    <HeadersTitle />
    <button @click="toggleCourseList">
      <i class="fas fa-tasks"></i> Tugas 1-7
    </button>
    <button @click="toggleToPost">
      <i class="fas fa-sticky-note"></i> {{ isPost ? 'Hide Post' : 'Show Post' }}
    </button>
    <button @click="toggleToDoFormVisibility">
      <i class="fas fa-list"></i> {{ isToDoFormVisible ? 'Hide Todos' : 'Show Todos' }}
    </button>

    <div v-if="showCourses" class="course-list">
      <h2 class="course-title">List of Tugas:</h2>
      <ul class="course-items">
        <li v-for="course in courses" :key="course.id" class="course-item">
          <a :href="getCourseUrl(course.id)" target="_self" class="course-link">
            <div class="course-info">
              <h3 class="course-name">{{ course.name }}</h3>
              <p class="course-details">{{ course.details }}</p>
            </div>
          </a>
        </li>
      </ul>
    </div>

    <section class="Post">
      <div v-if="isPost" class="container">
        <PostUser />
      </div>
    </section>

    <section class="Todos">
      <div v-if="isToDoFormVisible" class="container">
        <SlotCom />
        <div class="todo-actions">
          <input v-model="taskInput" type="text" placeholder="Add a task...">
          <button @click="addTask"><i class="fas fa-plus"></i> Add Task</button>
          <button @click="cancelTask" class="cancel"><i class="fas fa-ban"></i> Cancel</button>
          <button @click="clearAllTasks" class="clearalltask"><i class="fas fa-trash-alt"></i> Clear All Tasks</button>
          <button @click="deleteSelectedTasks" v-if="selectedTasks.length > 0"><i class="fas fa-trash-alt"></i> Delete Selected Tasks</button>
          <button @click="toggleFilterCompleted">
            <i class="fas fa-filter"></i> {{ filterCompleted ? 'Show Unfinished' : 'Show Finished' }}
          </button>
        </div>
        <div v-if="taskList.length === 0" class="no-tasks">No tasks available.</div>
        <div class="total-tasks">Total Tasks: {{ totalTasks }}</div>
        <ul>
          <li v-for="(task, index) in filteredTasks" :key="index" class="task-item" :class="{ selected: selectedTasks.includes(index) }">
            <input type="checkbox" v-model="selectedTasks" :value="index">
            <span :style="{ 'text-decoration': task.completed ? 'line-through' : 'none' }">{{ task.text }}</span>
            <input v-if="editing[index]" type="text" v-model="taskList[index].text" @keydown.enter="doneEditing(index)" @blur="doneEditing(index)">
            <input type="file" @change="handleImageUpload(index)">
            <img v-if="task.image" :src="task.image" alt="Task Image" class="task-image">
            <div class="button-group">
              <button v-if="!task.completed" @click="toggleEdit(index)">
                <i class="fas fa-edit"></i> {{ editing[index] ? 'Done' : 'Edit' }}
              </button>
              <button @click="deleteTask(index)"><i class="fas fa-trash-alt"></i> Delete</button>
              <button @click="toggleCompleted(index)">
                <i :class="task.completed ? 'fas fa-undo' : 'fas fa-check'"></i> {{ task.completed ? 'Mark as Not Finished' : 'Mark as Completed' }}
              </button>
              <button @click="viewTask(index)"><i class="fas fa-eye"></i> View</button>
            </div>
          </li>
        </ul>
        <div v-if="selectedTask !== null" class="task-details">
          <h2 class="task-title" style="color: #4caf50;">Data</h2>
          <p>{{ selectedTask.text }}</p>
          <img v-if="selectedTask.image" :src="selectedTask.image" alt="Task Image" class="selected-task-image">
          <button @click="selectedTask = null" class="close-button"><span style="font-size: 20px; color: black;">X</span></button>
        </div>
      </div>
    </section>
    <FooterMe />
  </section>
</template>
<script setup>
import { ref } from 'vue'
import PostUser from './components/UserPosts.vue'
import HeadersTitle from "./components/Header.vue"
import FooterMe from './components/Footer.vue'
import SlotCom from './components/HeaderTodos.vue'

const childMsg = ref('')
function ubahTeks(msg) {
  childMsg.value = msg
  alert(`Welcome To My website`)
}
</script>

<script>
export default {
  data() {
    return {
      taskInput: '',
      taskList: [],
      selectedTasks: [],
      editing: [],
      selectedTask: null,
      filterCompleted: false,
      isToDoFormVisible: false,
      isPost: false,
      courses: [
        { id: 1, name: 'Tugas 1', details: 'Lihat Tugas 1', url: 'https://ahmad-projectcv.netlify.app' },
        { id: 2, name: 'Tugas 2', details: 'Lihat Tugas 2', url: 'https://marifahmadn-223510434-lab-t2freetugas.netlify.app' },
        { id: 3, name: 'Tugas 3', details: 'Lihat Tugas 3', url: 'https://marifahmad.netlify.app' },
        { id: 4, name: 'Tugas 4', details: 'Lihat Tugas 4', url: 'https://marifahmad-223510434-prapbk.netlify.app' },
        { id: 5, name: 'Tugas 5', details: 'Lihat Tugas 5', url: 'https://marifahmad-223510434-ppbk-t5-landingp.netlify.app' },
        { id: 6, name: 'Tugas 6', details: 'Lihat Tugas 6', url: 'https://marifahmadnasution-223510434-t6-inapi.netlify.app' },
        { id: 7, name: 'Tugas 7', details: 'Lihat Tugas 7', url: 'https://marifahmadnasution-223510434-t7-pinia.netlify.app' }
      ],
      showCourses: false
    }
  },
  created() {
    const savedTasks = localStorage.getItem('taskList');
    if (savedTasks) {
      this.taskList = JSON.parse(savedTasks);
      this.editing = new Array(this.taskList.length).fill(false);
    }
  },
  watch: {
    taskList: {
      handler(newValue) {
        localStorage.setItem('taskList', JSON.stringify(newValue));
      },
      deep: true
    }
  },
  computed: {
    totalTasks() {
      return this.taskList.length;
    },
    filteredTasks() {
      return this.filterCompleted
        ? this.taskList.filter(task => task.completed)
        : this.taskList.filter(task => !task.completed);
    }
  },
  methods: {
    addTask() {
      if (this.taskInput.trim() !== '') {
        const newTask = { text: this.taskInput.trim(), image: null, completed: false };
        this.taskList.push(newTask);
        this.editing.push(false);
        this.taskInput = '';
      }
    },
    toggleCourseList() {
      this.showCourses = !this.showCourses;
    },
    getCourseUrl(courseId) {
      const course = this.courses.find(c => c.id === courseId);
      return course ? course.url : '#';
    },
    toggleToDoFormVisibility() {
      this.isToDoFormVisible = !this.isToDoFormVisible;
    },
    toggleToPost() {
      this.isPost = !this.isPost;
    },
    hideToDoForm() {
      this.isToDoFormVisible = false;
    },
    hidePost() {
      this.isPost = false;
    },
    showPost() {
      this.isPost = true;
    },
    showToDoForm() {
      this.isToDoFormVisible = true;
    },
    cancelTask() {
      this.taskInput = '';
    },
    deleteTask(index) {
      if (this.selectedTask === this.taskList[index]) {
        this.selectedTask = null;
      }
      this.taskList.splice(index, 1);
      this.editing.splice(index, 1);
    },
    clearAllTasks() {
      this.taskList = [];
      this.selectedTasks = [];
      this.editing = [];
    },
    deleteSelectedTasks() {
      this.selectedTasks.sort((a, b) => b - a);
      this.selectedTasks.forEach(index => {
        this.taskList.splice(index, 1);
        this.editing.splice(index, 1);
      });
      this.selectedTasks = [];
    },
    toggleEdit(index) {
      this.editing[index] = !this.editing[index];
    },
    doneEditing(index) {
      this.editing[index] = false;
    },
    handleImageUpload(index) {
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = () => {
        const imageDataUrl = reader.result;
        if (/^data:image\//.test(imageDataUrl)) {
          this.taskList[index].image = imageDataUrl;
        } else {
          alert('Please upload an image file.');
        }
      };
      reader.readAsDataURL(file);
    },
    toggleCompleted(index) {
      this.taskList[index].completed = !this.taskList[index].completed;
    },
    toggleFilterCompleted() {
      this.filterCompleted = !this.filterCompleted;
    },
    viewTask(index) {
      if (this.filterCompleted) {
        const completedTasks = this.taskList.filter(task => task.completed);
        this.selectedTask = { ...completedTasks[index], originalIndex: index };
      } else {
        const unfinishedTasks = this.taskList.filter(task => !task.completed);
        this.selectedTask = { ...unfinishedTasks[index], originalIndex: index };
      }
    }
  }
}
</script>
<style>
body {
  height: max-content;
  color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
  background: url('/src/assets/gunung.jpg') no-repeat center center fixed;
  background-size: cover;
}
</style>
<style scoped>
.course-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
  background:transparent;
    border: 2px solid rgb(255, 255, 255, .2);
    border-radius: 20px;
    backdrop-filter: blur(20px);
    box-shadow: 0 0 10px rgba(0, 0, 0, .2);; padding: 10px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.course-item {
  background-color: #fff;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}

.course-item:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
}

.course-link {
  display: block;
  text-decoration: none;
  color: inherit;
}

.course-info {
  padding: 15px;
}

.course-title {
  font-size: 1.5em;
  color: #fff;
  margin-bottom: 10px;
}

.course-name {
  font-size: 1.25em;
  color: #4caf50;
  margin-bottom: 5px;
}

.course-details {
  color: #666;
  font-size: 0.9em;
  line-height: 1.4;
}

.course-item img {
  width: 100%;
  height: auto;
  display: block;
}
.container {
  text-align: center;
  font-family: Arial, sans-serif;
  background-color: #d1f9d1;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

.footer {
  margin-top: 20px;
  background-color: #ffffffe6;
  color: #4caf50;
  box-shadow: #d1f9d1;
}

.footer a {
  color: rgb(75, 203, 16);
}

.footer a:hover {
  color: rgba(10, 250, 50, 0.664);
}

input[type="text"] {
  padding: 8px;
  margin-right: 8px;
}

input[type="checkbox"]:checked {
  background-color: #4caf50;
}

button {
  padding: 8px 16px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
  margin-left: 5px;
}

button:hover {
  background-color: #45a049;
}

ul {
  list-style-type: none;
  padding: 0;
}

.task-item {
  margin-top: 10px;
  background-color: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}

body {
  background-color: #d1f9f7f7;
}

.task-item.selected {
  transition: background-color 0.2s ease, transform 0.2s ease;
}

.task-item.selected:nth-child(odd) {
  background-color: #fff;
}

.task-item.selected:nth-child(even) {
  background-color: #f2f2f2;
}

.task-item.selected:hover {
  transform: scale(1.05);
}

.task-item input[type="text"] {
  padding: 8px;
}

.task-item img {
  margin-right: 5px;
}

.task-item input[type="file"] {
  margin-left: 5px;
}

.completed {
  text-decoration: line-through;
}

.button-group {
  margin-top: 5px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.button-group button {
  margin: 5px;
}

.button-group button:first-child {
  margin-right: auto;
}

.container button {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
  margin-left: 5px;
}

.container button i {
  margin-right: 8px;
}

.container button.cancel, .container button.clearalltask {
  margin-bottom: 5px;
}

.completed {
  text-decoration: line-through;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #fff;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 10px;
}

.task-item.selected {
  background-color: #f0f0f0;
}

.task-item img {
  max-width: 100px;
  max-height: 100px;
  margin-left: 10px;
}

.button-group {
  display: flex;
  justify-content: space-between;
}

.footer {
  background-color: #ffffffe6;
  color: #4caf50;
  box-shadow: #d1f9d1;
}

.footer a {
  color: rgb(75, 203, 16);
}

.footer a:hover {
  color: rgba(10, 250, 50, 0.664);
}

.todo-actions {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-bottom: 10px;
}

.todo-actions button {
  margin: 5px;
}

.todo-actions input[type="text"] {
  padding: 8px;
  margin-right: 10px;
  flex: 1;
}

.no-tasks {
  color: #4caf50;
  text-align: center;
  margin: 20px 0;
}

.total-tasks {
  text-align: center;
  font-weight: bold;
  margin: 10px 0;
}

.task-details {
  text-align: center;
  border-top: 1px solid #ddd;
  padding-top: 10px;
}

.selected-task-image {
  max-width: 200px;
  max-height: 200px;
  display: block;
  margin: 10px auto;
}

.close-button {
  background-color: #f44336;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 5px;
  margin-top: 10px;
}

.close-button:hover {
  background-color: #e53935;
}
</style>
