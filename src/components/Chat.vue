<script>

  import { Client } from '@stomp/stompjs';

  export default {
    name: 'ChatComponent',

    setup() {
      return {
        data: '',
        client: null,
        messages: []
      }
    },

    mounted() {
      this.connect();
    },

    methods: {

      connect() {
        this.client = new Client({
          brokerURL: 'ws://localhost:8080/wb-guide-websocket',
          connectHeaders: {},
          debug: (e) => {
            console.log(e);
          },
          onConnect: this.onConnect(),
          onWebSocketError: this.onWebSocketError(),
        });

        this.client.activate();

      },

      onConnect() {
          return () => {
            this.client.subscribe('/topic/messages', (message) => {
              const newMessage = JSON.parse(message.body);
              this.messages = [...this.messages, newMessage];
              this.scrollMessage();
            });
          };
      },

      onWebSocketError() {
        return (error) => {
          console.error('WebSocket Error: ', error);
        };
      },

      sendMessage() {
        this.client.publish({
          destination: "/app/socket",
          body: JSON.stringify({'content': this.data})
        });
      },

      scrollMessage() {
        this.$nextTick(() => {
          this.$forceUpdate();
          this.waitContainerExisting();
        });
      },

      waitContainerExisting() {
        setTimeout(() => {
            var chatContainer = this.$refs['chat'];
            if (chatContainer) {
              chatContainer.scrollTop = chatContainer.scrollHeight;
            }
          }, 500);
      }

    }
}

</script>

<template>
  <h2>Chat - Vitor Silva</h2>

  <div ref="chat" class="chat-container">

    <div :key="messageOrder"
      v-for="(message, messageOrder) in messages"
      :class="['chat-bubble', messageOrder % 2 === 0 ? 'chat-bubble-left' : 'chat-bubble-right']"
    >
      <p>{{ message.content }}</p>

    </div>

  </div>

   <!-- Área de input e botão -->

  <div class="chat-input-container">
      <input
        v-model="data"
        type="text"
        class="chat-input"
        @keydown.enter="sendMessage"
        placeholder="Digite uma mensagem..."
      />
      <button class="send-btn" @click="sendMessage">
        <i class="fa fa-paper-plane"></i>
      </button>
    </div>

</template>

<style scoped>

  h2 {
    text-align: center;
    font-family: 'Arial', sans-serif;
  }

  .chat-container {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    margin: 20px;
    max-width: 600px;
    max-height: 600px;
    margin: 20px auto;
    height: 70%;
    overflow-y: auto;
    padding-bottom: 10px;
    border-radius: 10px;
    background-color: #fff;
  }

  .chat-bubble {
    position: relative;
    padding: 10px 20px;
    max-width: 60%;
    border-radius: 20px;
    font-family: 'Arial', sans-serif;
    font-size: 16px;
    margin-bottom: 10px;
    display: inline-block;
    width: 50%;
  }

  .chat-bubble-left {
    background-color: #f1f1f1;
    align-self: flex-start;
  }

  .chat-bubble-right {
    background-color: #0078d4;
    color: white;
    align-self: flex-end;
    border-radius: 20px 20px 0 20px;
  }

  .chat-input-container {
    display: inline-block;
    position: absolute;
    border-radius: 20px;
    width: 95%;
    bottom: 0;
    background-color: #fff;
    padding: 10px;
  }

  .chat-input {
    width: 80%;
    padding: 10px;
    border-radius: 20px;
    background-color: #fff;
    font-size: 16px;
    color: #333;
    outline: none;
    margin-bottom: 25px;
  }

  /* Foco no campo de input */
  .chat-input:focus {
    border: 2px solid #0078d4;
  }

  /* Estilo do botão de envio */
  .send-btn {
    background-color: #0078d4;
    border: none;
    border-radius: 50%;
    padding: 10px;
    cursor: pointer;
    justify-content: center;
    align-items: center;
    transition: background-color 0.3s;
    margin-left: 4px;
    margin-bottom: 30px;
  }

  /* Ícone do botão de envio */
  .send-btn i {
    color: white;
    font-size: 20px;
  }

  /* Efeito de hover no botão */
  .send-btn:hover {
    background-color: #005a8d;
  }

</style>
