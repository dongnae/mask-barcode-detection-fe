<template>
  <md-content id="app" md-theme="primary" style="background: white; display: flex; vertical-align: center;">
    <img class="video" style="width: 60%" :src="img">
    <div style="border: 1px solid black; width: 40%;">
      <div style="display: flex; justify-content: space-around;">
        <h3>학번 {{ stu_num }}</h3>
        <h3>이름 {{ stu_name }}</h3>
      </div>
      <div style="display: flex; justify-content: space-around;">
        <h3>{{ mask_text }}</h3>
        <h3>{{ jaga_jindan_text }}</h3>
      </div>
      <div style="display: flex; justify-content: space-around;">
        <h3>출석 시간</h3>
        <h3>{{ (new Date).toLocaleString() }}</h3>
      </div>
      <div style="display: flex; justify-content: space-around;">
        <h3>온도</h3>
        <h3>정상</h3>
      </div>
      <div @click="test">test</div>
    </div>

    <md-dialog :md-active.sync="showRegisterDialog">
      <md-dialog-title>Preferences</md-dialog-title>

      <md-tabs md-dynamic-height>
        <md-tab md-label="학생 정보 등록">
          <md-field>
            <label for="name">바코드 번호</label>
            <md-input name="barcode" id="barcode" autocomplete="barcode" v-model="last_stu_data.barcode"/>
          </md-field>
          <md-field>
            <label for="name">이름</label>
            <md-input name="name" id="name" autocomplete="name" v-model="regis.name"/>
          </md-field>
          <md-field>
            <label for="num">학번</label>
            <md-input name="num" id="num" autocomplete="num" v-model="regis.num"/>
          </md-field>
          <md-field>
            <label for="birth">생일</label>
            <md-input name="birth" id="birth" autocomplete="birth" v-model="regis.birth"/>
          </md-field>
          <md-field>
            <label for="pw">자가진단 비밀번호</label>
            <md-input name="pw" id="pw" autocomplete="pw" v-model="regis.pw"/>
          </md-field>
        </md-tab>
      </md-tabs>
      <md-dialog-actions>
        <md-button @click="register">Close</md-button>
      </md-dialog-actions>
    </md-dialog>
  </md-content>
</template>

<script>
import {io} from 'socket.io-client';

let socket = io("ws://127.0.0.1:8000");

export default {
  name: 'App',
  data() {
    return {
      img: "",
      last_stu_data: {},
      submitted: "",
      jaga_jindan_status: -1,
      mask: -1,
      regis: {
        name: "",
        num: "",
        birth: new Date(),
        password: ""
      }
    }
  },
  computed: {
    showRegisterDialog() {
      return this.jaga_jindan_status === 1;
    },
    stu_name() {
      let name = this.last_stu_data.name;
      return name ?? "";
    },
    stu_num() {
      let num = this.last_stu_data.num;
      return num ?? "";
    },
    jaga_jindan_text() {
      if (this.jaga_jindan_status === -1) return "자가진단";
      else if (this.jaga_jindan_status === 0) return "자가진단 제출함";
      else if (this.jaga_jindan_status === 1) return "정보 등록";
      else return "자가진단 미제출";
    },
    mask_text() {
      if (this.mask === -1) return "마스크";
      else if (this.mask === 0) return "마스크 미착용";
      else return "마스크 착용";
    }
  },
  methods: {
    test() {
      /*socket.emit("register", {
        barcode: "바코드", num: "학번", name: "일번", birth: "생일", password: "비번"
      }, (res) => console.log(res));*/
      //socket.emit("submit", "C003000501", (res) => console.log(res));
    },
    reset() {
      this.regis = {
        name: "",
        num: "",
        birth: new Date(),
        password: ""
      };
    },
    register() {
      socket.emit("register", {
        barcode: this.last_stu_data.barcode,
        num: this.regis.num,
        name: this.regis.name,
        birth: this.regis.birth,
        password: this.regis.pw
      }, () => {
      });
    }
  },
  created() {
    socket.on("video", ret => this.img = `data:image/jpeg;base64, ${ret}`);
    socket.on("masked", ret => this.mask = ret);
    socket.on("jaga_jindan", ret => {
      //console.log(ret);
      if (ret.status === 1) {
        console.log("학생 정보 등록 필요");
      } else if (ret.status === 2) {
        this.last_stu_data = ret.data;
        console.log("자가진단 미제출");
      } else {
        this.last_stu_data = ret.data;
        console.log("자가진단 제출");
        //console.log(ret);
      }
      this.jaga_jindan_status = ret.status;
    });
  }
}
</script>

<style lang="scss">
@import "~vue-material/dist/theme/engine";

@include md-register-theme("primary", (
    primary: #ff4081,
    accent: #ff4081
));

@import "~vue-material/dist/theme/all";
</style>
