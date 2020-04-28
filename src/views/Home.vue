<template>
  <div class="home">
    <div id="local_stream"></div>
    <div id="remote_stream"></div>
  </div>
</template>

<script>
// @ is an alias to /src

import TRTC from "trtc-js-sdk";
import { genTestUserSig } from "@/assets/js/lib-generate-test-usersig.min.js";
export default {
  name: "Home",
  data() {
    return {
      userId: "user_" + parseInt(Math.random() * 100000000), //用户id --可更改
      roomId: 8888, //房间号--加入相同房间才能聊
      client: "", //客户端服务
      remoteStream: "", //远方播放流
      localStream: "" //本地流
    };
  },
  components: {},
  created() {
    const $t = this;
    this.$nextTick(() => {
      const { userId } = $t;
      //获取签名
      const config = genTestUserSig(userId);
      const sdkAppId = config.sdkAppId;
      const userSig = config.userSig;

      $t.client = TRTC.createClient({
        mode: "videoCall",
        sdkAppId,
        userId,
        userSig
      });

      $t.fun4start();
    });
  },
  methods: {
    //
    async fun4start() {
      this.fun4subStream();
      this.fun4join();
    },
    //创建发布本地音视频流
    async fun4createLocalStream() {
      const { userId } = this,
        $t = this;
      $t.localStream = TRTC.createStream({
        userId,
        audio: true,
        video: true
      });

      $t.localStream.setScreenProfile("120p");

      $t.localStream
        .initialize()
        .catch(error => {
          console.log(error, "----------------999");
          // console.error('初始化本地流失败 ' + error);
        })
        .then(() => {
          console.log("初始化本地流成功");
          $t.client
            .publish($t.localStream)
            .then(() => {
              console.log("本地流发布成功", $t.localStream.hasVideo());
            })
            .catch(e => {
              console.log(e);
            });
          console.log("本地播放成功");
          $t.localStream.play("local_stream");
        });
    },
    //本地视频流播放

    async fun4join() {
      const { roomId } = this,
        $t = this;
      $t.client
        .join({ roomId })
        .catch(error => {
          console.error("进房失败 " + error);
        })
        .then(() => {
          console.log("进房成功");

          $t.fun4createLocalStream();
        });
    },
    //订阅远端音视频流
    async fun4subStream() {
      const $t = this;
      $t.client.on("stream-added", event => {
        const remoteStream = event.stream;
        console.log("远端流增加: " + remoteStream.getId());
        //订阅远端流
        $t.client
          .subscribe(remoteStream, { audio: true, video: true })
          .catch(e => {
            console.error("failed to subscribe remoteStream", e);
          });
      });
      $t.client.on("stream-subscribed", event => {
        console.error(event, "---subscribed");
        const remoteStream = event.stream;
        const id = "remote_stream-" + remoteStream.getId();

        this.remoteStream = id;
        $t.remotePlay = remoteStream;
        // let dom = $t.$refs.remoteStream;
        // if(!dom)return;
        // //做了dom操作 需要使用$nextTick(),否则找不到创建的标签无法进行播放
        // console.log(666)
        // remoteStream.stop();

        console.error(
          "------555------",
          remoteStream.hasAudio(),
          remoteStream.hasVideo()
        );

        remoteStream
          .play("remote_stream")
          .then(() => {
            // autoplay success
          })
          .catch(e => {
            console.table(e);
          });
      });
    }

    //监听远端流
  }
};
</script>
<style lang="scss">
.home {
  #local_stream,
  #remote_stream {
    width: 100vw;
    height: 100px;
  }
}
</style>
