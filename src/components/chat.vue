<template>
  <div class="chat">
    <b-card title="Chat box" class="chat__box">
      <span v-if="userName">You name: {{userName}}</span>

      <div class="chat__message-wrap" v-chat-scroll>
        <div v-for="(message, index) in messages"
             :key="index"
             class="message"
             :class="{'message_another-user' : isAnotherUser(message)}">
          <b-badge>User: {{ message.user }} <br> {{ message.text }}</b-badge>
        </div>
      </div>
    </b-card>

    <b-card :title="formTitle" class="chat__form">
      <b-form v-if="!showForm" @submit="onStartChat">
        <b-form-group>
          <b-form-input v-model="userName" required placeholder="Enter name"></b-form-input>
        </b-form-group>

        <b-button type="submit" variant="primary">Start chat</b-button>
      </b-form>

      <b-form v-else @submit="onSubmit">
        <b-form-group>
          <b-form-textarea v-model="messageText" placeholder="Enter something..." rows="3" max-rows="6"></b-form-textarea>
        </b-form-group>

        <b-button type="submit" variant="primary">Submit</b-button>
      </b-form>
    </b-card>
  </div>
</template>

<script>
export default {
  name: 'Chat',
  data () {
    return {
      formTitle: 'Enter name',
      messageText: '',
      userName: '',
      showForm: false,
      messages: []
    }
  },
  mounted () {
    if (sessionStorage.getItem('name')) {
      this.getName()
    }

    if (localStorage.getItem('messages')) {
      this.getMessage()
    }

    window.onstorage = event => {
      if (event.key !== 'messages') {
        return
      }

      this.updateMessage(event)
    }
  },
  methods: {
    getName () {
      try {
        this.userName = sessionStorage.getItem('name')
        this.showForm = true
      } catch (e) {
        sessionStorage.removeItem('name')
      }
    },
    getMessage () {
      try {
        this.messages = JSON.parse(localStorage.getItem('messages'))
      } catch (e) {
        localStorage.removeItem('messages')
      }
    },
    onStartChat (event) {
      event.preventDefault()
      this.showForm = true
      this.formTitle = 'Enter something...'

      sessionStorage.setItem('name', this.userName)
    },
    onSubmit (event) {
      event.preventDefault()

      if (!this.messageText) {
        return
      }

      this.messages.push({text: this.messageText, user: this.userName})
      this.messageText = ''
      this.saveMessage()
    },
    isAnotherUser (message) {
      return message.user !== this.userName
    },
    saveMessage () {
      const parsed = JSON.stringify(this.messages)
      localStorage.setItem('messages', parsed)
    },
    updateMessage (event) {
      const newDate = JSON.parse(event.newValue)
      this.messages.push({text: newDate[newDate.length - 1].text, user: newDate[newDate.length - 1].user})
    }
  }
}
</script>
