<template>
  <section class="penggunaanEmit">
      <Title title="Emit" />
      <h1>{{ childMsg }}</h1>
      <!-- Conditionally render the EmitCom component based on childMsg value -->
      <EmitCom v-if="!childMsg" @response="ubahTeks" />
  </section>
  <section>
  <HeadersTitle />
  <button @click="toggleToPost">{{ isPost ? 'Hide Post' : 'Show Post' }}</button>
  <button @click="toggleToDoFormVisibility">{{ isToDoFormVisible ? 'Hide Todos' : 'Show Todos' }}</button><br><br>
  <section class="Post">
    <div v-if="isPost" class="container">
      <PostUser />
    </div>
  </section>
  <section class="Todos">
    <div v-if="isToDoFormVisible" class="container">
      <SlotCom />
      <input v-model="taskInput" type="text" placeholder="Add a task...">
      <button @click="addTask">Add Task</button>
      <button @click="cancelTask" class="cancel">Cancel</button>
      <button @click="clearAllTasks" class="clearalltask">Clear All Tasks</button>
      <button @click="deleteSelectedTasks" v-if="selectedTasks.length > 0">Delete Selected Tasks</button>
      <button @click="toggleFilterCompleted">{{ filterCompleted ? 'Show Unfinished' : 'Show Finished' }}</button>
      <div v-if="taskList.length === 0" style="color:#4caf50;">No tasks available.</div>
      <div class="t">Total Tasks: {{ totalTasks }}</div>
      <ul>
        <li v-for="(task, index) in filteredTasks" :key="index" class="task-item" :class="{ 'selected': selectedTasks.includes(index), 'completed': task.completed }">
          <input type="checkbox" v-model="selectedTasks" :value="index">
          <span v-if="!editing[index]" :style="{ 'text-decoration': task.completed ? 'line-through' : 'none' }">{{ task.text }}</span>
          <input v-else type="text" v-model="taskList[index].text" @keydown.enter="doneEditing(index)" @blur="doneEditing(index)">
          <input type="file" @change="handleImageUpload(index)">
          <img v-if="task.image" :src="task.image" alt="Task Image" style="max-width: 100px; max-height: 100px;">
          <div class="button-group">
            <button v-if="!task.completed" @click="toggleEdit(index)">
              {{ editing[index] ? 'Done' : 'Edit' }}
            </button>
            <button @click="deleteTask(index)">Delete</button>
            <button @click="toggleCompleted(index)">{{ task.completed ? 'Mark as Not Finished' : 'Mark as Completed' }}</button>
            <button @click="viewTask(index)">View</button>  
          </div>
        </li>
      </ul>
      <div v-if="selectedTask !== null">
        <h2 style="color: #4caf50;">Data</h2>
        <p>{{ selectedTask.text }}</p>
        <img v-if="selectedTask.image" :src="selectedTask.image" alt="Task Image" style="max-width: 200px; max-height: 200px;">
        <button @click="selectedTask = null"><span style="font-size: 20px; color: black;">X</span></button>
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
import EmitCom from './components/Emitbut.vue'

const childMsg = ref('')


function ubahTeks(msg) {
  childMsg.value = msg
  alert(`Welcome To My ebsite`)
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
      isPost: false
    }
  },
  created() {
    // Load taskList from local storage when component is created
    const savedTasks = localStorage.getItem('taskList');
    if (savedTasks) {
      this.taskList = JSON.parse(savedTasks);
    }
  },
  watch: {
    // Watch for changes in taskList and save to local storage
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
      if (this.filterCompleted) {
        return this.taskList.filter(task => task.completed);
      } else {
        return this.taskList.filter(task => !task.completed);
      }
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
      // Mengatur tampilan formulir to-do list ketika tombol ditekan
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
.t{
  color: #45a049;
}
h1{
  color: #45a049;
}
span{
  color: #45a049;
}
input[type="text"]{
  color: #45a049;
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
  background-color: #ffffffe6;
  color: #4caf50;
box-shadow: #d1f9d1;
  
}
.footer a{
  color: rgb(75, 203, 16);}
  .footer a:hover{
  color: rgba(10, 250, 50, 0.664);}
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
body{
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
.button-group{
  margin-top: 5px;
}
/* tablet, dll */
@media only screen and (max-width: 768px) {
  h1{
  color: #45a049;
}
span{
  color: #45a049;
}
input[type="text"]{
  color: #45a049;
}
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .container input[type="text"] {
    margin-bottom: 10px; /* Menambahkan margin bawah untuk input Add Task */
  }
  button.cancel{
    margin-top: 5px;
    margin-bottom: 5px;
  }
  button.clearalltask{
    margin-bottom: 5px;
  }
  
  .button-group {
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
}
</style>
