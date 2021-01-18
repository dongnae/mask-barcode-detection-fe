<template>
  <md-content id="app" md-theme="primary" style="background: white; display: flex; vertical-align: center;">
    <img class="video" style="width: 60%" :src="img">
    <div style="border: 1px solid black; width: 40%; font-size: 20px">
      <div style="display: flex; justify-content: space-around; border-bottom: 1px solid black;">
        <h3>학번: {{ stu_num }}</h3>
        <h3>이름: {{ stu_name }}</h3>
      </div>
      <div>
        <div style="display: flex; justify-content: space-around; border-bottom: 1px solid black;">
          <div
              style="width:30%; border-right: 1px solid black; text-align: center; display: flex; justify-content: space-around">
            <h3>마스크</h3>
            <img src="/마스크.PNG" style="height: auto; width: 100px;">
          </div>
          <div :style="mask_color" style="flex: 1; text-align: center;"><h3>{{ mask_text }}</h3></div>
        </div>
      </div>
      <div>
        <div style="display: flex; justify-content: space-around; border-bottom: 1px solid black;">
          <div
              style="width:30%; border-right: 1px solid black; text-align: center; display: flex; justify-content: space-around">
            <h3>자가진단</h3>
            <img src="/자가진단.PNG" style="height: 65px; width: auto;">
          </div>
          <div :style="jaga_jindan_color" style="flex: 1; text-align: center;"><h3>{{ jaga_jindan_text }}</h3></div>
        </div>
      </div>
      <div style="display: flex; justify-content: space-around; border-bottom: 1px solid black;">
        <div
            style="width:30%; border-right: 1px solid black; text-align: center; display: flex; justify-content: space-around">
          <h3>체온</h3>
          <img src="/온도계.PNG" style="height: auto; width: 50px;">
        </div>
        <div style="width:70%; background-color:#33FF00; text-align: center;"><h3>정상</h3></div>
      </div>
      <div style="font-size: 15px;text-align: center;">
        <h3>출석 시간 : {{ (new Date).toLocaleString() }}</h3>
      </div>
      <!--<div @click="test">test</div>-->
    </div>

    <md-dialog :md-active.sync="showRegisterDialog">
      <md-dialog-title>학생 정보 등록</md-dialog-title>

      <md-tabs md-dynamic-height>
        <md-tab md-label="학생 정보 등록">
          <md-field>
            <label for="name">바코드 번호</label>
            <md-input name="barcode" id="barcode" autocomplete="barcode" v-model="last_stu_data.barcode"
                      :disabled="true"/>
          </md-field>
          <md-field>
            <label for="name">이름</label>
            <md-input name="name" id="name" autocomplete="name" v-model="regis.name"/>
          </md-field>
          <md-field>
            <label for="num">학번</label>
            <md-input name="num" id="num" autocomplete="num" v-model="regis.num" type="number"/>
          </md-field>
          <md-field>
            <!--<md-datepicker v-model="regis.birth">
              <label>생일</label>
            </md-datepicker>-->
            <label for="birth">생일 (YYMMDD)</label>
            <md-input name="birth" id="birth" autocomplete="birth" type="number" v-model="regis.birthday"/>
          </md-field>
          <md-field>
            <label for="pw">자가진단 비밀번호</label>
            <md-input name="pw" id="pw" autocomplete="pw" v-model="regis.password" type="password" pattern="[0-9]*"
                      inputmode="numeric"/>
          </md-field>
        </md-tab>
      </md-tabs>
      <md-dialog-actions>
        <md-button @click="register">등록하기</md-button>
      </md-dialog-actions>
    </md-dialog>

    <md-dialog :md-active.sync="showSubmitDialog">
      <md-dialog-title>자가진단 제출</md-dialog-title>

      <md-tabs md-dynamic-height>
        <md-tab md-label="자가진단 제출">
          <div>
            <p><b>학생에게 코로나19가 의심되는 아래의 임상증상*이 있나요?</b><br>
              *(주요 임상증상) 기침, 호흡곤란, 오한, 근육통, 두통, 인후통, 후각·미각 소실 또는 폐렴<br>
              단, 기저질환 등으로 코로나19와 관계없이 평소에 다음 증상이 계속되는 경우는 제외<br>
              <b>학생 본인 또는 동거인이 방역당국에 의해 현재 자가격리가 이루어지고 있나요?</b><br>
              ※ &lt;방역당국 지침&gt; 최근 14일 이내 해외 입국자, 확진자와 접촉자 등은 자가격리 조치<br>
              단, 직업특성상 잦은 해외 입·출국으로 의심증상이 없는 경우 자가격리 면제<br>
            </p>
            <br>
            <p>
              위 중 해댱하는 사항이 하나라도 있나요?
            </p>
            <br>
            <md-radio v-model="form.yes" :value="true">예</md-radio>
            <md-radio v-model="form.yes" :value="false">아니요</md-radio>
          </div>
        </md-tab>
      </md-tabs>
      <md-dialog-actions>
        <md-button @click="submit">제출하기</md-button>
      </md-dialog-actions>
    </md-dialog>
    <md-snackbar md-position="center" :md-duration="2000" :md-active.sync="showRegisterSnackbar" md-persistent>
      <span>잘못된 정보입니다. 다시 적어주세요.</span>
      <md-button class="md-primary" @click="showRegisterSnackbar = false">Retry</md-button>
    </md-snackbar>
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
        birthday: "",
        password: ""
      },
      showRegisterSnackbar: false,
      form: {
        yes: false,
        no: false
      },
      //update: Date.now(),
      intervalId: 0,
      duration: 2000,
      requestOpen: false
    }
  },
  computed: {
    showRegisterDialog() {
      return this.jaga_jindan_status === 1 || this.jaga_jindan_status === 3;
    },
    showSubmitDialog() {
      return this.jaga_jindan_status === 2;
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
    jaga_jindan_color() {
      if (this.jaga_jindan_status === -1) return "background-color: white;";
      else if (this.jaga_jindan_status === 0) return "background-color: #33FF00;";
      else if (this.jaga_jindan_status === 1) return "background-color: gray;";
      else return "background-color: red;";
    },
    mask_text() {
      if (this.mask === -1) return "마스크";
      else if (this.mask === 0) return "마스크 미착용";
      else return "마스크 착용";
    },
    mask_color() {
      if (this.mask === -1) return "background-color: white;";
      else if (this.mask === 0) return "background-color: red;";
      else return "background-color: #33FF00;";
    },
  },
  methods: {
    test() {
      /*socket.emit("register", {
        barcode: "바코드", num: "학번", name: "일번", birth: "생일", password: "비번"
      }, (res) => console.log(res));*/
      //socket.emit("submit", "C003000501", (res) => console.log(res));
    },
    reset() {
      this.last_stu_data = {};
      this.jaga_jindan_status = -1;
      this.showRegisterSnackbar = false;
      this.regis = {
        name: "",
        num: "",
        birth: new Date(),
        password: ""
      };
      this.form = {
        yes: false,
        no: false
      };
    },
    register() {
      socket.emit("register", {
        barcode: this.last_stu_data.barcode,
        num: this.regis.num,
        name: this.regis.name,
        birth: this.regis.birthday,
        password: this.regis.password
      }, () => {
      });
    },
    submit() {
      socket.emit("submit", {
        barcode: this.last_stu_data.barcode,
      }, () => {
      });
    },
  },
  created() {
    socket.on("motor_close", () => {
      this.requestOpen = false;
      this.reset();
    });
    socket.on("video", ret => this.img = `data:image/jpeg;base64, ${ret}`);
    socket.on("masked", ret => this.mask = ret);
    socket.on("jaga_jindan", ret => {
      if (this.intervalId) clearInterval(this.intervalId);
      this.intervalId = 0;
      //console.log(ret);
      if (ret.status === 1) {
        //console.log("학생 정보 등록 필요");
      } else if (ret.status === 2) {
        //this.last_stu_data = ret.data;
        //console.log("자가진단 미제출");
      } else if (ret.status === 3) {
        this.showRegisterSnackbar = true;
        //this.last_stu_data = ret.data;
        //console.log("정보 incorrect");
      } else {
        //this.last_stu_data = ret.data;
        //console.log("자가진단 제출");
        //console.log(ret);
      }
      //ret.status = 1;
      let prev = this.showRegisterDialog;
      //this.update = Date.now();
      this.jaga_jindan_status = ret.status;
      if (!prev && this.showRegisterDialog) {
        this.reset();
      }
      //console.log(this.showRegisterDialog);
      if (ret.status === 3) this.regis = ret.data;
      this.last_stu_data = ret.data;
      console.log(ret, ret.data);

      if (ret.status === 0)
        this.intervalId = setTimeout(() => {
          console.log('reset');
          this.reset();
        }, 1000 * 5);
    });

    setInterval(() => {
      if (this.requestOpen) return;
      if (this.last_stu_data.name !== null && this.mask === 1 && this.jaga_jindan_status === 0) {
        this.requestOpen = true;
        console.log("open door");
        socket.emit("open");
      }
    }, 1000);
  }
}
</script>

<style>
@import url("https://cdn.jsdelivr.net/gh/moonspam/NanumBarunGothic@latest/nanumbarungothicsubset.css");

* {
  font-family: 'NanumBarunGothic', sans-serif;
}
</style>

<style lang="scss">
@import "~vue-material/dist/theme/engine";

@include md-register-theme("primary", (
    primary: #ff4081,
    accent: #ff4081
));

@import "~vue-material/dist/theme/all";
</style>
