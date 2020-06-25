<template>
  <el-container>
    <el-header>
      <img src="../../public/logo.png" class="logo" />
      <h1>Banking Assistant</h1>
    </el-header>
    <el-container>
      <el-aside>
        <img class="assistant" :src="assistant" />

        <el-button
          @click="changeAI()"
          type="success"
          icon="el-icon-refresh-left"
        ></el-button>
      </el-aside>
      <el-main>
        <div class="speech-bubble">
          <p>
            Xin chao ban, bay gio ban muon gi. Có khá nhiều khái niệm về quy
            trình tín dụng được đưa ra. Nhiều ý kiến cho rằng: Quy trình tín
            dung là tổng hợp các nguyên tắc, quy định của ngân hàng trong việc
            cấp tín dụng đối với khách hàng, bao gồm các công việc theo một
            trình tự nhất định kể từ khi bắt đầu cho đến khi chấm dứt quan hệ
            tín dụng. Đây là một qúa trình bao gồm nhiều giai đoạn mang tính
            chất liên hoàn, theo một trật tự nhất định, đồng thời có quan hệ
            chặt chẽ và gắn bó với nhau
          </p>
        </div>
        <div>
          <img
            v-if="recording"
            class="voice-effect"
            src="../../public/sound.gif"
          />
        </div>
        <div>
          <el-button
            @click="startRecording()"
            type="primary"
            v-if="!recording"
            icon="el-icon-microphone"
            round
            >Record</el-button
          >
          <el-button
            v-if="recording"
            icon="el-icon-turn-off-microphone"
            type="danger"
            round
            @click="stopRecording()"
            >Record</el-button
          >
        </div>
        <div>
          <div v-if="question !== null" class="speech-bubble2">
            <p>
              {{ question }}
            </p>
          </div>
        </div>
      </el-main>
    </el-container>
    <el-footer>
      <el-row>
        <el-col :span="12"> <p>Hotline: 19006969</p> </el-col>
        <el-col :span="12"></el-col>
      </el-row>
    </el-footer>
  </el-container>
</template>

<script>
import AIAssistant from "../../public/AI.png";
import redAIAssistant from "../../public/redAI.png";
import greenAI from "../../public/greenAI.png";
import SiriWave from "siriwave";

import axios from "axios";
var audio_context;
var recorder;
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },

  data() {
    return {
      assistant: AIAssistant,
      assistantArray: [AIAssistant, redAIAssistant, greenAI],
      recording: false,
      question: null,
    };
  },

  methods: {
    startUserMedia(stream) {
      const input = audio_context.createMediaStreamSource(stream);
      // Media stream created
      recorder = new Recorder(input);
      // Recorder initialised
    },
    startRecording() {
      this.question = null;
      this.recording = true;
      // Start Recording
      recorder && recorder.record();
      this.result = false;
      this.btn = false;
      this.btnStop = true;
      setTimeout(this.stopRecording, 58000);
    },
    stopRecording() {
      // Stopped Recording
      recorder && recorder.stop();
      this.btnStop = false;
      this.loader = true;
      // create WAV download link using audio data blob
      this.processRecording();
      this.recording = false;
      this.question = "This is my question";
      recorder.clear();
    },
    processRecording() {
      const vm = this;

      recorder &&
        recorder.exportWAV(function(blob) {
          var reader = new window.FileReader();
          reader.readAsDataURL(blob);
          reader.onloadend = () => {
            const baseData = reader.result;
            const base64Data = baseData.replace("data:audio/wav;base64,", "");
            vm.data.audio.content = base64Data;
            vm.data.config.languageCode = vm.selected;
            axios
              .post(
                `https://speech.googleapis.com/v1/speech:recognize?key=${vm.apiKey}`,
                vm.data
              )
              .then((response) => {
                const result = response.data.results[0].alternatives[0];
                vm.textResult = result.transcript;
                vm.btn = true;
                vm.loader = false;
                vm.result = true;
              })
              .catch((error) => {
                vm.loader = false;
                vm.resultError = true;
                console.log("ERROR:" + error);
              });
          };
        });
    },
    redirectError() {
      window.location.href = "http://localhost:8080/";
    },
  },
  created() {
    try {
      window.AudioContext = window.AudioContext || window.webkitAudioContext;
      navigator.getUserMedia =
        navigator.getUserMedia || navigator.webkitGetUserMedia;
      window.URL = window.URL || window.webkitURL;

      audio_context = new AudioContext();
      console.log("Audio context set up.");
      console.log(
        "navigator.getUserMedia " +
          (navigator.getUserMedia ? "available." : "not present!")
      );
    } catch (e) {
      alert("No web audio support in this browser!");
    }
    const that = this;
    navigator.getUserMedia(
      {
        audio: true,
      },
      function(stream) {
        that.startUserMedia(stream);
      },
      function(e) {
        console.log("No live audio input: " + e);
      }
    );
  },
  changeAI: function() {
    this.assistant = this.assistantArray[
      Math.floor(Math.random() * this.assistantArray.length)
    ];
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-header {
  height: 120px !important;
  text-align: center;
  background: #1a7bbc;
  color: white;
  font-size: 30px;
}
.el-aside {
  width: 300px;
  height: auto;
}
.el-footer {
  height: 100px;
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
  background: #1a7bbc;
  color: white;
  text-align: center;
  font-size: 30px;
}
.voice-effect {
  margin: 30px;
  border-radius: 100px;
  width: 200px;
  height: 200px;
}

.assistant {
  width: 300px;
}
.speech-bubble {
  position: relative;
  background: #00bda7;
  border-radius: 0.4em;
  height: auto;
  width: auto;
  word-wrap: break-word;
  padding: 60px 20px;
  margin: 1em 0;
  text-align: left;
  color: white;
  font-weight: bold;
  text-shadow: 0 -0.05em 0.1em rgba(0, 0, 0, 0.3);
}

.speech-bubble:after {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  width: 0;
  height: 0;
  border: 25px solid transparent;
  border-right-color: #00bda7;
  border-left: 0;
  border-bottom: 0;
  margin-top: -12.5px;
  margin-left: -25px;
}
.speech-bubble2 {
  position: relative;
  background: #c329e2;
  border-radius: 0.4em;
  height: auto;
  width: auto;
  word-wrap: break-word;
  padding: 60px 20px;
  margin: 1em 0;
  text-align: left;
  color: white;
  font-weight: bold;
  text-shadow: 0 -0.05em 0.1em rgba(0, 0, 0, 0.3);
}

.speech-bubble2:after {
  content: "";
  position: absolute;
  right: 0;
  top: 50%;
  width: 0;
  height: 0;
  border: 20px solid transparent;
  border-left-color: #c329e2;
  border-right: 0;
  border-bottom: 0;
  margin-top: -10px;
  margin-right: -20px;
}
.logo {
  width: 100px;
  height: 100px;
  float: left;
  padding-top: 10px;
}
</style>
