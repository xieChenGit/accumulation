<template>
  <el-dialog
    title="录制"
    :visible.sync="dialogVisible"
    width="30%"
    :before-close="handleClose"
  >
    <div class="tip">{{ tips }}</div>
    <div class="wrapper-box">
      <video
        ref="video"
        class="video"
        width="200"
        @click="showVideo(false)"
        @touchend.prevent="showVideo(false)"
      ></video>
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button @click="handleClose">取 消</el-button>
      <el-button type="primary" @click="handleClose">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
import { randomFileName } from "@/utils/file";
import { upload } from "@/api/file";

export default {
  props: {
    switch: {
      type: Boolean,
    },
    phoneNum: {
      type: [String, Number],
    },
    id: {
      type: [String, Number],
    },
  },
  data() {
    return {
      dialogVisible: this.switch,
      recorder: null,
      url: "",
      chunks: [],
      video: null,
      second: 0,
      time: null,
      tips: "请准备...",
    };
  },
  methods: {
    handleClose() {
      this.$confirm("确认关闭？")
        .then((_) => {
          this.$emit("update:changeSwitch", false);
        })
        .catch((_) => {});
    },
    // 初始化
    requestAudioAccess() {
      navigator.mediaDevices
        .getUserMedia({
          audio: true,
          video: true,
        })
        .then(
          (stream) => {
            this.recorder = new window.MediaRecorder(stream);
            this.stream = stream;
            this.bindEvents();

            this.onMousedown();
          },
          (error) => {
            alert("出错，请确保已允许浏览器获取音视频权限");
          }
        );
    },
    // 绑定事件
    bindEvents() {
      this.recorder.ondataavailable = this.getRecordingData;
      this.recorder.onstop = this.saveRecordingData;
    },
    // 手指按下
    onMousedown() {
      this.showVideo(true);
      this.onPreview();
      this.onStart();
      this.time = setInterval(() => {
        this.second++;
        if (this.second >= 3) {
          this.tips = "请眨眼";
        }
        if (this.second >= 6) {
          this.tips = "请点头";
        }
        if (this.second >= 10) {
          this.tips = "正在提交，请稍后...";
          clearInterval(this.time);

          this.onMouseup();
        }
      }, 1000);
    },
    // 手指松开
    onMouseup() {
      this.onStop();
      this.showVideo(false);
    },
    //
    onPreview() {
      this.video.srcObject = this.stream;
      this.video.muted = true;
      this.video.play();
    },
    // 显示视频
    showVideo(bShow) {
      if (bShow) {
        this.video.style.display = "block";
      } else {
        this.video.style.display = "none";
        this.video.pause();
      }
    },
    // 开始录制
    onStart() {
      this.recorder.start();
    },
    // 结束录制
    onStop() {
      this.recorder.stop();
    },
    // 开始播放
    onPlay(index) {
      this.showVideo(true);
      this.video.src = this.url;
      this.video.muted = false;
      this.video.play();

      this.bindAudioEvent();
    },
    // 绑定事件
    bindAudioEvent() {
      this.video.onended = () => {
        this.showVideo(false);
      };
    },
    getRecordingData(e) {
      this.chunks.push(e.data);
    },
    // 保存数据
    saveRecordingData() {
      let blob = new Blob(this.chunks, {
          type: "video/webm",
        }),
        videoStream = URL.createObjectURL(blob);
      this.url = videoStream;
      this.chunks = [];

      let fileName = `${randomFileName(10)}.mp4`;
      let file = new window.File([videoStream], fileName, {
        type: "mp4",
      });

      if (!this.dialogVisible) return;
      this.upload(file);
    },
    // 上传视频
    upload(data) {
      //   let { phoneNum, id } = this.$parent.form;

      let formData = new FormData();
      formData.append("file", data);
      formData.append("phoneNum", this.phoneNum);
      formData.append("id", this.id);

      upload(formData).then((res) => {
        this.tips = "录制成功！";
        this.$emit("uploadVideoSuccess", res);
      });
    },
  },
  mounted() {
    if (!navigator.mediaDevices) {
      alert("您的浏览器不支持获取用户设备");
      return;
    }
    if (!window.MediaRecorder) {
      alert("您的浏览器不支持录音");
      return;
    }
    this.$nextTick(() => {
      this.video = this.$refs["video"];
      this.requestAudioAccess();

      //   setTimeout(() => {}, 1000);
    });
  },
};
</script>

<style>
.wrapper-box {
  width: 300px;
  height: 300px;
  margin: 0 auto;
  border-radius: 50%;
  border: 1px solid #333333;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}
.video {
  width: 400px;
  height: 400px;
}
.tip {
  font-size: 20px;
  color: red;
  text-align: center;
  margin: 10px 0;
}
</style>