<template>
    <div>
      <h1>User Posts</h1>
      <div class="form-group">
        <label for="userSelect">Select User: </label>
        <select class="form-control" id="userSelect" v-model="selectedUser" @change="fetchUserPosts">
          <option value="" disabled selected>Select User</option>
          <option v-for="user in users" :value="user.id" :key="user.id">~{{ user.name }}</option>
        </select>
      </div>
      <table style="margin-top: 10px;" class="table" v-if="userPosts.length > 0">
        <thead class="thead-dark">
          <tr>
            <th style="border: 1px solid black; padding: 5px;"  scope="col">No</th>
            <th style="border: 1px solid black; padding: 5px;" scope="col">Title</th>
            <th style="border: 1px solid black ; padding: 5px;" scope="col">Post Content</th>
          </tr>
        </thead>
        <tbody style="border: 1px solid black;  padding: 5px;">
          <tr v-for="(post, index) in userPosts" :key="post.id">
            <th style="border: 1px solid black;  padding: 5px;" scope="row">{{ index + 1 }}</th>
            <td style="border: 1px solid black;  padding: 5px;">{{ post.title }}</td>
            <td style="border: 1px solid black;">{{ post.body }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>Loading...</p>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        users: [],
        selectedUser: null,
        userPosts: []
      };
    },
    mounted() {
      this.fetchUsers();
    },
    methods: {
      async fetchUsers() {
        try {
          const response = await fetch('https://jsonplaceholder.typicode.com/users');
          if (!response.ok) {
            throw new Error('Failed to fetch users');
          }
          const data = await response.json();
          this.users = data;
        } catch (error) {
          console.error(error);
        }
      },
      async fetchUserPosts() {
        try {
          const response = await fetch(`https://jsonplaceholder.typicode.com/posts?userId=${this.selectedUser}`);
          if (!response.ok) {
            throw new Error('Failed to fetch user posts');
          }
          const data = await response.json();
          this.userPosts = data;
        } catch (error) {
          console.error(error);
        }
      }
    }
  };
  </script>

 
  <style scoped>
  /* Gaya untuk custom form-control */
  .custom-select {
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    background-color: #099120;
    border: 1px solid #ced4da;
    border-radius: 0.25rem;
    padding: 50px;
    font-size: 1rem;
    cursor: pointer;
  }
  
  .custom-select:focus {
    outline: none;
    border-color: #4d90fe;
  }
  .table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1rem;
}


  
  </style>