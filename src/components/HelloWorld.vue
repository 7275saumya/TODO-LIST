<template>
  <div class="todo-app">
    <h1 class="app-title">To-Do List</h1>
    <div class="input-container">
      <input type="text" v-model="newTask" class="task-input" placeholder="Add New Task" @keyup.enter="addTask" maxlength="35">
      <button @click="addTask" class="add-btn">Add</button>
    </div>
    <ul class="task-list">
      <li v-for="task in tasks" :key="task.id" class="task-item">
        <input type="checkbox" v-model="task.completed" @change="updateTask(task)" class="task-checkbox" title="Mark as complete">
        <span v-if="!task.editing" class="task-text" :class="{ 'completed-text': task.completed }">{{ task.title }}</span>
        <input v-else type="text" v-model="task.title" class="task-edit-input" @keyup.enter="saveEditedTask(task)" @blur="cancelEdit(task)" maxlength="35">
        <span class="task-date">{{ formatDate(task.created_at) }}</span>
        <button @click="editTask(task)" class="edit-btn" title="Edit"><i class="fas fa-pencil-alt"></i></button>
        <button @click="deleteTask(task.id)" class="delete-btn" title="Delete"><i class="fas fa-trash-alt"></i></button>
        <button v-if="task.completed" @click="reAddTask(task.id)" class="re-add-btn" title="Re-add"><i class="fas fa-undo"></i></button>
      </li>
    </ul>
  </div>
</template>

<script>
import { getAllTasks, createTask, updateTask, deleteTask } from '../api';

export default {
  name: 'HelloWorld',
  data() {
    return {
      tasks: [],
      newTask: '',
      backgroundColor: '#72d6cf', // Background color
    };
  },
  async created() {
    await this.fetchTasks();
  },
  computed: {
    incompleteTasks() {
      return this.tasks.filter(task => !task.completed);
    }
  },
  methods: {
    async fetchTasks() {
      const response = await getAllTasks();
      this.tasks = response.data.map(task => ({ ...task, editing: false }));
    },
    async addTask() {
      if (this.newTask.trim() === '') return;
      const newTask = {
        title: this.newTask,
        completed: false,
        created_at: new Date().toISOString(),
      };
      await createTask(newTask);
      this.newTask = '';
      await this.fetchTasks();
    },
    async updateTask(task) {
      await updateTask(task.id, task);
      await this.fetchTasks();
    },
    async deleteTask(id) {
      await deleteTask(id);
      await this.fetchTasks();
    },
    editTask(task) {
      task.editing = true;
    },
    async saveEditedTask(task) {
      task.editing = false;
      await this.updateTask(task);
    },
    cancelEdit(task) {
      task.editing = false;
    },
    async reAddTask(id) {
      const task = this.tasks.find(task => task.id === id);
      task.completed = false;
      await this.updateTask(task);
    },
    formatDate(dateString) {
      const date = new Date(dateString);
      const options = {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        second: 'numeric',
        hour12: true // Use 12-hour format
      };
      return date.toLocaleString('en-US', options);
    }
  }
};
</script>

<style scoped>
.todo-app {
  max-width: 600px; /* Adjust as needed */
  margin: 0 auto;
  padding: 20px;
  border: 1px transparent;
  border-radius: 5px;
  font-family: Times New Roman,Arial, sans-serif;
  background-color: #fff;
}

.app-title {
  margin-bottom: 20px;
  text-align: center;
  color: black;
}

.input-container {
  display: flex;
  margin-bottom: 20px;
}

.task-input {
  flex: 1;
  padding: 8px;
  border: 1px solid black;
  border-radius: 3px 0 0 3px;
}

.add-btn {
  padding: 8px 15px;
  border: none;
  border-radius: 0 3px 3px 0;
  background-color: #4caf50;
  color: white;
  cursor: pointer;
}

.task-list {
  list-style-type: none;
  padding: 0;
}

.task-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.task-checkbox {
  margin-right: 10px;
}

.task-text {
  flex: 2; /* Adjust width as needed */
  color: black;
}

.task-date {
  width: 200px; /* Fixed width */
  margin-right: 10px;
  color: black;
}

.edit-btn,
.delete-btn,
.re-add-btn {
  padding: 5px 10px;
  border: none;
  border-radius: 3px;
  color: black;
  cursor: pointer;
  margin-right: 5px; /* Add margin to create space between buttons */
}

.edit-btn {
  background-color: transparent; /* Remove background color */
}

.delete-btn,
.re-add-btn {
  background-color: transparent; /* Remove background color */
}

.completed-text {
  text-decoration: line-through;
}
</style>
