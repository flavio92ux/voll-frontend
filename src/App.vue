<template>
    <fieldset>
      <div class="topnav">
        <span id="user">Welcome to VueJs chat! Your nickname is {{ nick }} </span>
        <div class="search-container">
          <form>
            <input
              id="nickInput"
              type="text"
              placeholder="Change your nickname"
              v-model="newNick"
            >
            <button
              v-on:click="changeNick($event)"         
            >
            Submit
            </button>
          </form>
        </div>
      </div>
    </fieldset>
    <div class="main-container">
      <aside class="message-container">
        <ul class="messages">
          <li
            class="messages"
            id="messages"
            v-for="(comment, index) in comments" :key="index"
          >
          {{ comment }} 
          </li>
        </ul>
      </aside>
      <aside className="users-container">
        <ul>OnLine Users</ul>
          <li
            v-for="(user, index) in users" :key="index"
          >
            {{ user }}
          </li>
      </aside>
    </div>
    <form class="form" id="form" action="">
      <input
        class="input"
        id="input"
        autocomplete="off"
        v-model="message"
      />
      <button
        v-on:click="addComment($event)"
      >
        Send
      </button>
    </form>
</template>

<script>
import { io } from 'socket.io-client';

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      comments: [],
      users: [],
      nick: '',
      newNick: '',
      message: '',
      socket: io(process.env.VUE_APP_SOCKET_ENDPOINT),
    };
  },
  created() {
    this.socket.on('addNewUser', (myUser) => {
      if (!this.nick) {
        this.nick = myUser.nickname;
        sessionStorage.setItem('nickname', myUser.nickname);
      }
    });

    this.socket.on('refreshList', (listUser) => {
      this.users = []
      const mainUser = sessionStorage.getItem('nickname');
      this.users.push(mainUser);

      listUser.forEach((item) => {
        if (item.nickname != mainUser) this.users.push(item.nickname)
      });
    });

    console.log(process.env.VUE_APP_SOCKET_ENDPOINT);

    fetch(process.env.VUE_APP_SOCKET_ENDPOINT)
      .then((res) => res.json())
      .then((data) => {
        data.forEach((item, index) => {
          if (index >= data.length - 30) {
            const msg = `${item.moment} - ${item.nick}: ${item.message}`
            this.comments.push(msg);
          }
        });
      });

    this.socket.on('message', (message) => {
      const comments = this.comments;
      comments.push(message)
      this.comments = comments.splice(comments.length - 30)
    });

  },
  methods: {
    addComment(e) {
      e.preventDefault();
      if (this.message) {
        this.socket.emit('message', { chatMessage: this.message, nickname: this.nick })
      }
      this.message = '';
    },
    changeNick(e) {
      e.preventDefault();
      if (this.newNick) {
        const oldUser = sessionStorage.getItem('nickname');
        const newUser = this.newNick;
        sessionStorage.setItem('nickname', newUser);
        this.socket.emit('replaceUser', { oldUser, newUser });
        this.nick = newUser;
        this.newNick = '';
      }
    }
  },
  beforeUnmount() {
    this.socket.disconnect();
  }
}
</script>

<style scoped>
body {
  font-family: -apple-system,
    BlinkMacSystemFont, Roboto, Helvetica, Arial, sans-serif;
  margin: 0;
  padding-bottom: 3rem;
}

.main-container {
  display: flex;
  justify-content: space-between;
}

.container {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
}

.users-container {
  width: 20%;
}

.message-container {
  position: sticky;
  width: 95%;
}

.form {
  backdrop-filter: blur(10px);
  background: rgba(0, 0, 0, 0.15);
  bottom: 0;
  box-sizing: border-box;
  display: flex;
  height: 3rem;
  left: 0;
  padding: 0.25rem;
  position: fixed;
  right: 0;
}

.input {
  border: none;
  border-radius: 2rem;
  flex-grow: 1;
  margin: 0.25rem;
  padding: 0 1rem;
}

.input:focus {
  outline: none;
}

.form > button {
  background: #333;
  border: none;
  border-radius: 3px;
  color: white;
  margin: 0.25rem;
  outline: none;
  padding: 0 1rem;
}

.messages {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.messages > li {
  padding: 0.05rem
}

.messages > li:nth-child( odd ) {
  background: #efefef;
}

fieldset {
  padding: 0%;
}

.topnav {
  overflow: hidden;
  background-color: #e9e9e9;
}

.topnav span {
  float: left;
  display: block;
  color: black;
  text-align: center;
  padding: 14px 16px;
  text-decoration: none;
  font-size: 17px;
}

.topnav .search-container {
  float: right;
}

.topnav input[type=text] {
  padding: 6px;
  margin-top: 8px;
  font-size: 17px;
  border: none;
}

.topnav .search-container button {
  float: right;
  padding: 6px;
  margin-top: 8px;
  margin-right: 16px;
  background: #ddd;
  font-size: 17px;
  border: none;
  cursor: pointer;
}

.topnav .search-container button:hover {
  background: #ccc;
}

@media screen and (max-width: 600px) {
  .topnav .search-container {
    float: none;
  }
  .topnav span, .topnav input[type=text], .topnav .search-container button {
    float: none;
    display: block;
    text-align: left;
    width: 100%;
    margin: 0;
    padding: 14px;
  }
  .topnav input[type=text] {
    border: 1px solid #ccc;  
  }
}

</style>