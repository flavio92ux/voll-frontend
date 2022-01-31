<template>
    <fieldset class="nick">
      <span id="user">Welcome to Trybe WebChat! Your nickname is </span>
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
      <ul id="online-users">OnLine Users</ul>
      <ul class="messages" id="messages">
        <li>teste</li>
      </ul>
    </aside>
    <form class="form" id="form" action="">
      <input
        class="input"
        id="input"
        autocomplete="off"
        data-testid="message-box"
      />
      <button data-testid="send-button">Send</button>
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
      data: ['loren', 'ipson'],
    };
  },
  created() {
    let socket = io(process.env.VUE_APP_SOCKET_ENDPOINT);

    socket.emit('my message', 'Hello there from Vue.');

    socket.on('my broadcast', (data) => {
      this.data = data;
    });
  },
  methods: {
    addComment() {
      console.log("ola")
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