<!--
** 红包兑换
-->

<template>
  <div class="put-wrapper" :class="{'active':isShowRob||isShowCode}">
    <title-bar title="红包兑换"/>
    <header class="put-header" :class="{'active':isShowRob||isShowCode}">
      <p class="put-money"><span class="price">{{parseInt(userInfo.redpacket)/100}}</span> <span class="unit">元</span>
      </p>
    </header>
    <div class="center">
      <h2 class="sub-title">请选择兑换金额</h2>
      <ul class="box">
        <li class="item-money" v-for="item in list" :key="item.id" @click="selectItem(item)">
          <div class="select-icon"><img
            :src="item.total_convert?item.is_select?require('img/envelopes/item_yes_bg.png'):require('img/envelopes/item_bg.png'):require('img/envelopes/item_no_bg.png')"
            alt="">
          </div>
          <p class="wrap">{{item.money/100}}元</p>
          <p class="text">{{item.is_prompt?'限时兑换':'不限时兑换'}}</p>
        </li>
      </ul>
      <div class="submit-btn" @click="submitClick">
        <img src="~/img/envelopes/submit_btn.png" alt="">
      </div>
      <p class="desc-test">兑换码成功生成后，请关注微信公众号“德趣通科技”输入获取的兑换码，领取微信红包</p>
    </div>
    <transition-scale v-if="isShowRob">
      <div class="rob-wrapper">
        <div class="center">
          <div class="close" @click="()=>{isShowRob = false}"><img src="~/img/com_img/close.png" alt=""></div>
          <h3 class="title">限时兑换 <span class="money">{{selected.money/100}}元</span></h3>
          <div class="qiang-btn" @click="gotoEbvelop"><img src="~/img/envelopes/qiang.png" alt=""></div>
          <p class="limit">今日份额：<span class="orange">{{selected.convert_num}}/{{selected.total_convert}}</span></p>
          <div class="rule-box" v-if="selected.is_prompt">
            <p>规则说明</p>
            <br>
            <p v-html="descContent(selected.description)"></p>
          </div>
        </div>
      </div>
    </transition-scale>
    <transition-scale v-if="isShowCode">
      <div class="code-wrapper">
        <div class="center">
          <div class="close" @click="()=>{isShowCode = false}"><img src="~/img/com_img/close.png" alt=""></div>
          <p class="text code">您的兑换码：
            <span class="orange">{{formData.conv_code}}</span>
            <span @click="copyCode" class="copy-text">点击我复制</span>
          </p>
          <p class="text desc" ref="copyText">请关注微信公众号“德趣通科技”领取。</p>
          <p class="text desc">兑换明细可以在红包明细中查看。</p>
          <textarea style="position: absolute;width: 0;height: 0;opacity: 0" ref="codeText"
                    v-model="formData.conv_code"></textarea>
        </div>
      </div>
    </transition-scale>
  </div>
</template>

<script>
  import {mapGetters} from "vuex";
  import TitleBar from "@/common/TitleBar";
  import TransitionScale from "@/common/TransitionScale";
  import api from "../../api/BaseService";
  import {showToast, updateBaseInfo} from "../../common/util/Utils";

  export default {
    name: "PutForward",
    data() {
      return {
        list: [],         //兑换列表
        selected: {},     //选中的数据
        formData: [],     //抢到后的数据
        isShowRob: false, //是否展示抢红包弹窗
        isShowCode: false,//是否展示抢到红包结果弹窗
        oneClick: true,   //禁止重复点击
      }
    },
    mounted() {
      this._getRedPacketRules();
    },
    methods: {
      async _getRedPacketRules() {
        let result = await api.getRedPacketRules();
        result.data.forEach((v) => {
          v.is_select = false;
        });
        this.list = result.data;
      },
      selectItem(item) {
        if (!item.total_convert) {
          return;
        }
        if (!item.is_select) {
          let arr = this.list;
          arr.forEach((v) => {
            v.is_select = false;
            if (v.id === item.id) {
              v.is_select = true;
            }
          })
        }
      },
      //提交兑换金额
      submitClick() {
        this.list.forEach((v) => {
          if (v.is_select) {
            this.selected = v;
          }
        });
        if (this.selected.hasOwnProperty('id')) {
          this.isShowRob = true;
        }
      },
      //抢名额
      async gotoEbvelop() {
        if (this.oneClick) {
          this.onclick = false;
          let result = await api.postRedPacketForm(this.selected.id);
          if (result.code == 1) {
            this.formData = result.data;
            this.isShowRob = false;
            this.isShowCode = true;
            updateBaseInfo();
            this._getRedPacketRules();
          } else {
            this.isShowRob = false;
            this.isShowCode = false;
            showToast(result.msg, 'cancel', 1000);
          }

          setTime(() => this.oneClick = true, 500);
        }
      },
      //点击我复制
      copyCode() {
        let codeText = this.$refs.codeText;
        codeText.select();
        let copyText = document.execCommand('Copy');
        if (copyText) {
          showToast('复制成功');
        }
      },
      descContent(value) {
        return value.replace(/\n/g, '<br />');
      },
    },
    computed: {
      ...mapGetters(['userInfo'])
    },
    components: {TitleBar, TransitionScale}
  }
</script>

<style scoped lang="less">
  @import "~assets/style/index.less";

  .put-wrapper {
    &.active {
      position: fixed;
      top: 0;
      left: 0;
      bottom: 0;
      right: 0;
      .background-url("~img/envelopes/select_bg.png");
    }
    .put-header {
      display: flex;
      justify-content: center;
      align-items: center;
      width: 100%;
      height: 220px;
      .background-url('~img/envelopes/header_bg.png');
      &.active {
        background-image: none;
      }
      .put-money {
        margin-top: -50px;
        color: @whiteColor;
      }
      .price {
        font-size: 70px;
        font-weight: 600;
      }
      .unit {
        font-size: @maxFontSize;
      }
    }
    .center {
      padding: 30px;
      padding-bottom: 0;
      .sub-title {
        display: inline-block;
        margin-bottom: 40px;
        padding: 10px 18px;
        font-size: @maxFontSize;
        background-color: @mainColor;
        color: @whiteColor;
        -webkit-border-radius: 12px;
        -moz-border-radius: 12px;
        border-radius: 12px;
      }
      .box {
        width: 100%;
        display: flex;
        flex-wrap: wrap;
        color: @whiteColor;
      }
      .item-money {
        margin: 16px 10px;
        position: relative;
        .select-icon {
          width: 206px;
          height: 96px;
          .img-spread;
        }
        .wrap {
          position: absolute;
          top: 16px;
          width: 100%;
          height: 30px;
          line-height: 30px;
          text-align: center;
          font-size: @maxFontSize;
          font-weight: 600;
        }
        .text {
          position: absolute;
          top: 22+38px;
          width: 100%;
          text-align: center;
          font-size: @minFontSize;
        }
      }
      .submit-btn {
        margin: 80px auto 40px;
        width: 404px;
        height: 80px;
        .img-spread;
      }
      .desc-test {
        margin: 0 auto;
        width: 660px;
        font-size: @subFontSize;
        line-height: 40px;
        color: @grayColor;
        text-align: center;
        word-break: break-all;
      }
    }
  }

  .rob-wrapper, .code-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: @bgOpacity;
    .close {
      position: absolute;
      top: -40/2px;
      right: -40/2px;
      width: 60px;
      height: 60px;
      .img-spread;
    }
    .orange {
      color: orange;
    }
  }

  .rob-wrapper {
    .center {
      position: relative;
      width: 574px;
      height: 698px;
      .background-url('~img/envelopes/rob-bg.png');
      .title {
        text-align: center;
        line-height: 90px;
        .money {
          color: #F1FE01;
        }
      }
      .qiang-btn {
        margin: 0 auto;
        width: 160px;
        height: 160px;
        .img-spread;
      }
      .limit {
        line-height: 100px;
        text-align: center;
      }
      .rule-box {
        margin-top: 20px;
        line-height: 40px;
      }
    }
  }

  .code-wrapper {
    .center {
      position: relative;
      width: 578px;
      height: 414px;
      .background-url('~img/envelopes/code_bg.png');
      .text {
        font-size: @mainFontSize;
        text-align: center;
      }
      .code {
        margin-top: 60px;
        line-height: 100px;
        .orange {
          font-size: @maxFontSize;
          font-weight: 600;
        }
        .copy-text {
          padding: 3px 6px;
          font-size: @minFontSize;
          color: @whiteColor;
          background-color: @grayColor;
        }
      }
      .desc {
        line-height: 50px;
      }
    }
  }
</style>
