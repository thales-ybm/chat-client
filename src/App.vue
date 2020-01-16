<template>
  <div id="app">
    <chat-room />
  </div>
</template>

<script>
import ChatRoom from './components/chat-room.vue';

export default {
  name: 'App',
  components: {
    ChatRoom
  }
};
</script>
