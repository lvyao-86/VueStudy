<!-- BKmv-mini页 -->
<template>
  <div class="voice header">
    <yd-layout>
      <yd-navbar slot="navbar" :title="'语音识别'">
        <router-link to="/main" slot="left">
          <yd-navbar-back-icon></yd-navbar-back-icon>
        </router-link>
        <yd-button
          slot="right"
          class="btn"
          @click.native="isShow = true"
        ></yd-button>
      </yd-navbar>

      <div
        v-for="(item, index) in recorderList"
        :key="index"
        class="recorderstyle"
      >
        <!-- <yd-button @click="_playRecord(index)">{{ item.size }}</yd-button> -->
        <yd-button @click.native="_playRecord(index)">{{
          item.size
        }}</yd-button>
        <yd-button @click.native="_downloadRecord(index)">Download</yd-button>
      </div>

      <!-- <div v-for="(item, index) in recorderList" :key="index">
        <span>{{ item.size }}<yd-button>Download</yd-button></span>
      </div> -->
      <!-- <yd-list theme="5"> -->
      <!-- 这里可选1/2/3/4/5五种样式  -->
      <!-- <yd-list-item v-for="(item, index) in recorderList" :key="index">
          <img slot="img" :src="item.img">
          <span slot="title">{{ item.size }}</span>
          <yd-list-other slot="other">
                <div>
                    <span class="demo-list-price"><em>¥</em>{{item.price}}</span>
                    <span class="demo-list-del-price">¥{{item.w_price}}</span>
                </div>
                <div>content</div>
            </yd-list-other>
        </yd-list-item>
      </yd-list> -->
      <!-- <div>
        <button disabled="disabled" style="float:right;margin:0.9rem 1.3rem 0" @touchstart="showTime()">语音识别开启</button>
      </div> -->
      <div class="voice-bottom">
        <span class="tooshort"
          ><img src="../../static/images/icon-tooshort.jpg"
        /></span>
        <div class="voice-text">
          <p class="voice-text-model1 textnone">
            {{ textState }}
            <span class="dot-ani"></span>
          </p>
          <p class="voice-text-model2 textnone">无常.云解析中</p>
          <p class="voice-text-model3 textnone">{{ textConent }}</p>
        </div>
        <span class="say">
          <p>按住说话</p>
        </span>
        <div class="voice-change ">
          <div class="voice-right">
            <div class="voice-rightcircle"></div>
          </div>
          <div class="voice-left">
            <div class="voice-leftcircle"></div>
          </div>
          <div
            class="voice-outer"
            @touchstart.prevent="voiceBegin()"
            @touchend.prevent="voiceEnd()"
          >
            <img src="../../static/images/icon-voice.png" />
          </div>
        </div>
        <div class="voice-down">
          <span class="voice-down-leftimg " @touchstart="hideBottom()"
            ><img src="../../static/images/icon-down.png"
          /></span>
          <span class="voice-down-lefttext textnone " @click="clearTable()"
            >清空</span
          >
          <span class="voice-down-righttext textnone"></span>
        </div>
      </div>
    </yd-layout>
  </div>
</template>
<script>
// import func from './vue-temp/vue-editor-bridge'
export default {
  name: 'Vzhinengfenxi',
  data: function () {
    return {
      textState: '正在录音',
      textConent: '',
      beginTime: null,
      endTime: null,
      timeout: null,
      recorder: null,
      recorderList: []
    }
  },
  methods: {
    voiceBegin: function () {
      this.beginTime = new Date()
      this.textConent = ''
      $('.tooshort').removeClass('tooshort-hide')
      $('.say').addClass('texthidden')
      $('.voice-change').addClass('begin-shrink')
      $('.voice-down-leftimg,.voice-text-model3').addClass('textnone')
      $('.voice-text-model1').removeClass('textnone')
      this._startRecording()
    },
    voiceEnd: function () {
      this.endTime = new Date()
      if (this.endTime - this.beginTime < 1000) {
        $('.tooshort').addClass('tooshort-hide')
        $('.say').removeClass('texthidden')
        $('.voice-change').removeClass('begin-shrink')
        $('.voice-down-leftimg,.voice-text-model3').removeClass('textnone')
        $('.voice-text-model1,.voice-down-lefttext').addClass('textnone')
        this._stopRecord()
      } else {
        $('.voice-change')
          .addClass('begin-circle')
          .removeClass('begin-shrink')
        $('.voice-text-model1').addClass('textnone')
        $('.voice-text-model2').removeClass('textnone')
        // 没有服务器 直接保存到本地
        // this._uploadRecord()
        // 保存到本地 并直接播放
        this._saveRecord()
        this.timeout = setTimeout(function () {
          $('.voice-text-model2').addClass('textnone')
          $(
            '.voice-text-model3,.voice-down-lefttext,.voice-down-righttext'
          ).removeClass('textnone')

          $('.voice-change').removeClass('begin-circle')
        }, 2000)
      }
    },
    clearTable: function () {
      $(
        '.voice-down-lefttext,.voice-down-righttext,.voice-text-model3'
      ).addClass('textnone')
      $('.say').removeClass('texthidden')
      $('.voice-down-leftimg').removeClass('textnone')
    },
    showTime: function () {
      // $('.voice-bottom').addClass('showTime').removeClass("hide-bottom")
      // $('button').addClass('texthidden')
    },
    hideBottom: function () {
      // $('.voice-bottom').removeClass('showTime').addClass("hide-bottom")
      // $('button').removeClass('texthidden')
    },
    _startRecording: function () {
      let _this = this
      HZRecorder.get(function (rec) {
        //console.log(rec)
        _this.recorder = rec

        _this.recorder.start()
      })
    },
    _stopRecord: function () {
      let _this = this
      _this.recorder.start()
    },
    _uploadRecord: function () {
      let _this = this
      //上传录音地址
      //  _this.recorder.upload('../../static/h5voice/save.php', function(
      _this.recorder.upload('https://XXX.cn/h5voice/save.php', function (
        state,
        e
      ) {
        switch (state) {
          case 'uploading':
            break
          case 'ok':
            _this.textConent = JSON.parse(e.target.response).data
            //console.log(_this.textConent.data)
            break
          case 'error':
            alert('上传失败')
            break
          case 'cancel':
            alert('上传被取消')
            break
        }
      })
    },
    _saveRecord: function () {
      let _this = this
      //上传录音地址
      this.recorderList.push(_this.recorder.save())
      // _this.recorder.save()
    },
    _downloadRecord: function (index) {
      let _this = this
      //上传录音地址
      // this.recorderList.push(_this.recorder.save())
      _this.recorder.download(this.recorderList[index])
      // _this.recorder.save()
    },
    _playRecord: function (index) {
      let _this = this
      //上传录音地址
      // console.log("点击")
      console.log(this.recorderList[index])
      _this.recorder.play1(this.recorderList[index])
      // debugger
      // this.recorderList.push(_this.recorder.save())
      // _this.recorder.save()
    }
  },
  computed: {},

  mounted () { },
  updated () { },
  destroyed () {
    clearTimeout(this.timeout)
  }
}
</script>
<style>
.recorderstyle {
  padding: 20px;
}
</style>

