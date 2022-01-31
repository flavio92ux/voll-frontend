<template>
    <fieldset class="nick">
      <span id="user">Welcome to Trybe WebChat! Your nickname is {{ nick }} </span>
      <div>
        <input
          id="nickInput"
          type="text"
          placeholder="Change your nickname"
        >
        <button
          id="nickBtn"
          data-testid="nickname-button"
        >
        Submit
        </button>
      </div>
    </fieldset>
    <aside className="container-users">
      <div id="online-users">OnLine Users</div>
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
      nick: '',
      message: '',
    };
  },
  created() {
    let socket = io(process.env.VUE_APP_SOCKET_ENDPOINT);

    socket.on('addNewUser', (nick) => {
      if (!this.nick) {
        this.nick = nick;
        sessionStorage.setItem('nickname', nick);
      }
    });

    socket.on('message', (message) => {
      this.comments.push(message)
    });

    socket.on('my broadcast', (data) => {
      this.data = data;
    });
  },
  methods: {
    addComment(e) {
      e.preventDefault();
      let socket = io(process.env.VUE_APP_SOCKET_ENDPOINT);
      if (this.message) {
        socket.emit('message', { chatMessage: this.message, nickname: this.nick })
      }
      this.message = '';
    }
  },
  beforeUnmount() {
    let socket = io(process.env.VUE_APP_SOCKET_ENDPOINT);
    socket.disconnect();
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