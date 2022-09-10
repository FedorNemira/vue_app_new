<template>
  <div>
    <form @submit.prevent>
      <div class="row form-group ml-1">
        <h4 id="show">Add post</h4>
        <div class="col-8 mb-3"> 
          <input v-bind:value="username" @input="username = $event.target.value" class="form-control" type="text" placeholder="Username">
        </div>
        <div class="col-8 mb-3"> 
          <input v-bind:value="email" @input="email = $event.target.value" class="form-control" type="text" placeholder="Email">
        </div>
        <div class="col-8 mb-3"> 
          <input v-bind:value="password" @input="password = $event.target.value" class="form-control" type="password" placeholder="Password">
        </div>
        <div class="col-8 mb-3"> 
          <button class="btn btn-primary pull-right" @click="createUser">Create User</button>
        </div>
      </div>
    </form>
    <div class="post" v-for="user in users" :key="user.id">  
      <div><strong>User: </strong> {{ user.username }}</div>
      <div><strong>Email: </strong> {{ user.email }}</div>
      <div><strong>Active: </strong> {{ user.active }}</div>
    </div>
  </div>   
</template>

<script>
  import { Centrifuge } from 'centrifuge';
  
  export default {  
    data() {
      return {
        users: [],
        username: '',
        email: '',
        password: '',
        active: '',
      }
    },
    methods: {
      loadUsers() {
        fetch(`${process.env.VUE_APP_API_URL}/register/`, {
          method: 'GET',
        })
        .then(response => response.json())
        .then(data => {
          this.users = data
        })
      },
      createUser() {
        const info = {
          'username': this.username,
          'email': this.email,
          'password': this.password,
        }
        fetch(`${process.env.VUE_APP_API_URL}/register/`, {
          method: 'POST',
          body: JSON.stringify(info),
        })
        .then(response => response.json())
        .then(data => {
          if (data.success) {
            let newUser= {
              id: Date.now(),
              username:  this.username,
              email: this.email,
              active: data.active,
            }
            this.users.push(newUser);
          }
        });
      },
    },
    mounted(){
      this.loadUsers()

      const centrifuge = new Centrifuge("ws://194.67.110.10:8000/connection/websocket", {
        token: process.env.VUE_APP_CENTRIFUGO_TOKEN,
        debug: true
      });

      const container = document.getElementById("show");

      centrifuge.on('connecting', function (ctx) {
        console.log(`connecting: ${ctx.code}, ${ctx.reason}`);
      }).on('connected', function (ctx) {
        console.log(`connected over ${ctx.transport}`);
      }).on('disconnected', function (ctx) {
        console.log(`disconnected: ${ctx.code}, ${ctx.reason}`);
      }).connect();

      const sub = centrifuge.newSubscription("channel");

      sub.on('publication', function (ctx) {
        container.innerHTML = ctx.data.title;
      }).on('subscribing', function (ctx) {
        console.log(`subscribing: ${ctx.code}, ${ctx.reason}`);
      }).on('subscribed', function (ctx) {
        console.log('subscribed', ctx);
      }).on('unsubscribed', function (ctx) {
        console.log(`unsubscribed: ${ctx.code}, ${ctx.reason}`);
      }).subscribe();

    },  
  }
</script>

<style>
  * {
    margin: 1em;
    padding: 0;
    box-sizing: border-box;
  }
  .post {
    padding: 15px;
    border: 3px solid teal; 
    margin-top: 10px;

  }
  button {
    margin-top: 10px;
  }

</style>
