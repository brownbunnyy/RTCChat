<template>
<el-container style="height: 100vh">
    <el-header style="font-family:Pacifico">Our Chaaaaaat!!</el-header>
    <el-container>
        <el-aside width="250px">
          <p>Members</p>
          <ul v-if="room" style="list-style: none">
            <li>{{my_peer_id}} (you)</li>
            <li v-for="(member,index) in room.members" :key="index">{{member}}</li>
          </ul>
        </el-aside>
        <el-main v-if="room">
            <el-row>
                <el-col :span="24">
                    <div class="block">
                        <el-timeline style="overflow:auto; height:60vh">
                            <el-timeline-item v-for="(message ,index) in messages" :key="index" :id="message.id" class="infinite-list-item" :timestamp="message.time" placement="top">
                                <el-card>
                                    <h4>{{message.peer_id}}</h4>
                                    <p>{{message.message}}</p>
                                </el-card>
                            </el-timeline-item>
                        </el-timeline>
                    </div>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="23" :offset="1">
                    <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="Please input" v-model="local_message" />
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="23" :offset="1">
                    <el-button @click="sendMessage">send</el-button>
                </el-col>
            </el-row>
        </el-main>
        <el-main v-else>
            <el-row>
                <el-col :span="10">
                    <p>Join Room</p>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="2">
                    room name
                </el-col>
                <el-col :span="10">
                    <el-input type="text" v-model="room_id" />
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="10" :offset="2">
                    <el-button @click="makeRoom">join</el-button>
                </el-col>
            </el-row>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
export default {
    data() {
        return {
            my_peer_id: '',
            name: '',
            room_id: 'test',
            room: null,
            messages: [],
            local_message: '',
        }
    },
    mounted: function () {
        const Peer = require("skyway-js");

        this.peer = new Peer({
            key: process.env.API_KEY,
            debug: 3,
        });

        this.peer.on('open', () => {
            this.my_peer_id = this.peer.id;
        });

    },
    methods: {
        makeRoom: function () {
            const room = this.peer.joinRoom(this.room_id, {
                mode: 'sfu',
            });

            this.room = room;

            room.once('open', () => {
                this.pushMessage(this.peerId, 'You joined');
            });

            room.once('close', () => {
                this.pushMessage(this.peerId, 'You lefted');
            });

            room.on('peerJoin', peerId => {
                this.pushMessage(peerId, `${peerId} joined`);
            });

            room.on('peerLeave', peerId => {
                this.pushMessage(peerId, `${peerId} lefted`);
            });

            room.on('data', ({
                data,
                src
            }) => {
                this.pushMessage(src, data);
            });
        },
        sendMessage: function () {
            this.room.send(this.local_message);
            this.pushMessage(this.my_peer_id, this.local_message);
            this.local_message = '';
        },
        pushMessage: function (peer_id, message) {
            const id = `message-${this.messages.length}`;
            this.messages.push({
                id: id,
                peer_id: peer_id,
                time: this.now(),
                message: message,
            });
            this.$nextTick(() => {
                document.getElementById(id).scrollIntoView({
                    behavior: 'smooth'
                });
            });
        },
        now: function () {
            let dd = new Date();
            return `${dd.getHours()}:${dd.getMinutes()}:${dd.getSeconds()}`;
        }
    },
}
</script>
