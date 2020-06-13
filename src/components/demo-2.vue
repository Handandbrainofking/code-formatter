<template>
  <div class="device-wrapper">
    <scroller class="scroll-view"
              :style="{ paddingBottom: fitBottomSize }"
              :show-scrollbar="false"
              :offset-accuracy="20"
              @scroll="pageScrollHandler">
      <div class="device">
        <div class="device-dashboard">
          <info-panel title="设置温度"
                      indicator=20
                      :desc-group="panelDescGroup"
                      :status-code="deviceStatusCode"
                      :base-config="baseConfig"
                      text-color="#ffffff">
          </info-panel>
        </div>
        <dof-catalog title="标题为中文信息面板"
                     :has-margin="true"></dof-catalog>
        <div class="device-dashboard">
          <info-panel title="设置温度"
                      indicator="中档"
                      :desc-group="panelDescGroup"
                      :status-code="deviceStatusCode"
                      :base-config="baseConfig"
                      unit=""
                      text-color="#ffffff">
          </info-panel>
        </div>
        <dof-catalog title="无统计数据的信息面板"
                     :has-margin="true"></dof-catalog>
        <div class="device-dashboard">
          <info-panel title="设置温度"
                      indicator=20
                      :status-code="deviceStatusCode"
                      :base-config="baseConfig"
                      text-color="#ffffff">
          </info-panel>
        </div>
        <dof-catalog title="无统计数据无状态的信息面板"
                     :has-margin="true"></dof-catalog>
        <div class="device-dashboard">
          <info-panel title="设置温度"
                      indicator=20
                      :status-code="deviceStatusCode"
                      :base-config="simpleConfig"
                      text-color="#ffffff">
          </info-panel>
        </div>
      </div>
      <div v-if="false"
           class="button-list">
        <dof-button type="primary"
                    text="场景一"
                    @dofButtonClicked="btnClicked(1)"></dof-button>
        <dof-button type="primary"
                    text="场景二"
                    @dofButtonClicked="btnClicked(2)"></dof-button>
        <dof-button type="primary"
                    text="场景三"
                    @dofButtonClicked="btnClicked(3)"></dof-button>
        <dof-button type="primary"
                    text="场景四"
                    @dofButtonClicked="btnClicked(4)"></dof-button>
      </div>
    </scroller>

    <dof-minibar :style="{ position: 'fixed', top: 0 }"
                 :title="deviceName"
                 text-color="#ffffff"
                 :background-color="minibarTransitionBg"
                 :leftButton="leftButton"
                 :right-button="moreIcon"
                 :hasBottomBorder="false"
                 @dofMinibarRightButtonClicked="minibarRightButtonClick"></dof-minibar>
  </div>
</template>

<script>
import { DofMinibar, Utils, DofButton, DofCatalog } from "dolphin-weex-ui";
import infoPanel from "src/components/digital-indicator-panel";
import { ASSETS_DATA_WARMER as ASSETS_DATA } from "./type.js";
const modal = weex.requireModule("modal");

export default {
  components: { DofMinibar, infoPanel, DofButton, DofCatalog },
  data: () => ({
    deviceName: "主视信息",
    leftButton: "/assets/image/header/back_white@3x.png",
    moreIcon: "/assets/image/header/icon_more@3x.png",
    //Y方向上滚动的距离
    scrollDistanceY: 0,
    deviceStatusCode: 3,
    showIndex: 2,
    panelDescGroup: [
      {
        title: "室内温度",
        value: "24°",
        style: {
          color: "#ffffff",
          fontSize: "24px"
        }
      },
      {
        title: "室内温度",
        value: "24°",
        style: {
          color: "#ffffff",
          fontSize: "24px"
        }
      },
      {
        title: "室内温度",
        value: "24°",
        style: {
          color: "#ffffff",
          fontSize: "24px"
        }
      }
    ],
    baseConfig: ASSETS_DATA
  }),
  created() {
    this.isIPhoneX = Utils.env.isIPhoneX();
  },
  computed: {
    navHeight() {
      let result = "20";
      if (weex.config.env.statusBarHeight) {
        if (weex.config.env.platform === "iOS") {
          // iOS使用pt为单位
          result = weex.config.env.statusBarHeight;
        } else {
          // 安卓使用px为单位
          result = weex.config.env.statusBarHeight / weex.config.env.scale;
        }
      }
      return (+result + 44) * 2;
    },
    minibarTransitionBg() {
      let opacityValue =
        (1 / this.navHeight) * Math.min(this.scrollDistanceY, this.navHeight) <
        0.2
          ? 0
          : (1 / this.navHeight) *
            Math.min(this.scrollDistanceY, this.navHeight);
      return `rgba(38, 122, 255, ${opacityValue})`;
    },

    fitBottomSize() {
      return this.isIPhoneX ? "256px" : "188px";
    },

    simpleConfig() {
      let { baseConfig } = this;
      return { "3": { ...baseConfig[3], text: "" } };
    }
  },
  mounted() {},
  methods: {
    minibarRightButtonClick() {
      let moreUrl = "meijv-template-page-more.js";
      this.$push(moreUrl);
      // this.$reload()
    },

    btnClicked(index) {
      console.log(index);
      // this.$toast(index)
      this.showIndex = index;
      this.$toast(this.showIndex);
    },

    pageScrollHandler(e) {
      let {
        contentOffset: { y }
      } = e;
      this.scrollDistanceY = Math.abs(y);
    }
  },
  watch: {}
};
</script>

<style lang="scss" scoped>
@import "src/css/dolphinweex", "src/css/plugin";
.device-wrapper {
  background-color: #f9f9f9;
  // background-image: linear-gradient(to bottom, #267aff, #00c6fb);
}
.scroll-view {
  padding-bottom: 300px;
}

.device {
  padding-bottom: 16px;
  &-dashboard {
    width: 750px;
    min-height: 600px;
    &-main-bg {
      &-png {
        width: 600px;
        height: 600px;
      }
    }
  }
  &-action-board {
    padding: 0 32px;
  }
}
.button-list {
  height: 450px;
  margin: -30px auto;
  justify-content: space-between;
  align-items: center;
  /* padding: 32px 0; */
}
</style>
