<template>
    <fieldset class="nick">
      <span id="user">Welcome to VueJs chat! Your nickname is {{ nick }} </span>
      <div>
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
      </div>
    </fieldset>
    <aside className="container-users">
      <div>OnLine Users</div>
      <div
        v-for="(user, index) in users" :key="index"
      >
        {{ user }}
      </div>
      <div
        class="messages"
        id="messages"
        v-for="(comment, index) in comments" :key="index"
      >
       {{ comment }} 
      </div>
    </aside>
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

    fetch('http://localhost:3000/')
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
    changeNick() {
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

.container {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
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
  padding: 0.5rem 1rem;
}

.messages > li:nth-child( odd ) {
  background: #efefef;
}
</style>