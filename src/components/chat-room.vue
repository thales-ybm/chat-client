<template>
  <div class="hello">
    <div v-if="state > 0" class="login">
      <p>
        <input v-model="name" placeholder="输入姓名" />
      </p>
      <p>
        <button value="登陆" @click="onLogin">
          登陆
        </button>
      </p>
    </div>
    <div v-else class="chat-room">
      <div class="room">
        <div class="name">
          {{ room }}
        </div>
        <div class="tips">
          请选择说话对象
        </div>
        <div class="users">
          <div class="user" @click="onAllCheck">
            全体
          </div>
          <template v-for="(item, index) in users">
            <div v-if="name==item" :key="index" class="user">
              {{ item }}(你)
            </div>
            <div v-else :key="index" class="user" :data-name="item" @click="onUserCheck">
              {{ item }}
            </div>
          </template>
        </div>
      </div>
      <div class="message">
        <textarea v-model="message" placeholder="请输入消息" />
        <span v-if="target==''">全体</span>
        <span v-else>{{ target }}</span>
        <button value="发送" @click="onSend">
          发送
        </button>
      </div>
      <div class="board">
        <p v-for="(item, index) in board" :key="index" :class="{ 'board-msg':true, self:item.self }">
          <span>
            {{ item.msg }}
          </span>
          <span>
            ({{ timeFormat(item.time) }})
          </span>
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client';

export default {
  name: 'ChatRoom',
  data() {
    return {
      name: '',
      state: 1,
      message: '',
      room: '',
      target: '',
      users: [],
      board: []
    };
  },
  socket: null,
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const socket = io('http://localhost:4001', {
        autoConnect: false,
        reconnection: false
      });
      socket.on('connect', e => {
        socket.emit('login', this.name);
      });
      socket.on('room', e => {
        let msg = `“${e.room}”聊天室`;
        if (e.type === 'join') msg = '进入了' + msg;
        else msg = '离开了' + msg;
        if (e.self) {
          this.state = 0;
          msg = '你' + msg;
          this.room = e.room;
        } else {
          msg = e.user + msg;
        }
        this.users = e.users;
        e.msg = msg;
        this.cost(e);
      });
      socket.on('talk', e => {
        console.log(e);
        let msg = e.from === this.name ? '你' : e.from;
        if (e.to) msg += '悄悄对' + (e.to === this.name ? '你' : e.to);
        msg += '说' + e.msg;
        e.msg = msg;
        this.cost(e);
      });
      socket.on('system', e => {
        alert(e);
      });
      this.socket = socket;
    },
    /**
     * 链接
     */
    connect() {
      this.socket.open();
    },
    timeFormat(time) {
      const d = new Date(time);
      return `${d.getHours()}:${d.getMinutes()}:${d.getSeconds()}`;
    },
    /**
     * 消息版发布
     */
    cost(obj) {
      this.board.unshift(obj);
    },
    /**
     * 登陆
     */
    onLogin() {
      this.connect();
    },
    onAllCheck(e) {
      this.target = '';
    },
    onUserCheck(e) {
      this.target = e.currentTarget.dataset.name;
    },
    /**
     * 发送
     */
    onSend() {
      const obj = {
        msg: this.message
      };
      if (this.target) obj.to = this.target;
      this.socket.emit('talk', obj);
      this.message = '';
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.login {
  max-width: 300px;
  max-height: 200px;
  padding: 10px;
  margin: 10px auto;
  border: #333 solid thin;
}
.room {
  max-width: 300px;
  max-height: 200px;
  padding: 10px;
  margin: 10px 0;
  border: #333 solid thin;
}
.room .name {
  font-size: 1rem;
}
.room .tips {
  font-size: 0.8rem;
}
.room .users {
  display: inline;
  overflow: scroll;
}
.room .users .user {
  display: inline-block;
  padding: 1px 5px;
}
.message {
  max-width: 300px;
  max-height: 200px;
  padding: 10px;
  margin: 10px 0;
  border: #333 solid thin;
}
.board {
  max-width: 500px;
  max-height: 500px;
  padding: 10px;
  border: #333 solid thin;
  overflow: scroll;
}
.board .board-msg {
  border-bottom: #333 dashed thin;
}
.board .board-msg.self {
  background-color: aqua;
}
</style>
