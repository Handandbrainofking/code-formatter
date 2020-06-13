<template>
  <div class="device">
    <scroller
      class="device-scroll-view"
      :style="{ paddingBottom: fitBottomSize }"
      :show-scrollbar="false"
      :offset-accuracy="20"
      @scroll="pageScrollHandler"
    >
      <indicator-dashboard
        title="实际温度"
        :indicator="deviceNumber"
        :status-code="deviceStatusCode"
        :desc-group="panelDescData"
        :base-config="baseConfig"
      ></indicator-dashboard>
      <div class="device-action-board">
        <dof-cell
          avatar-icon="http://dolphin-weex-dev.msmartlife.cn/cdn/images/business/other/calorifier_icon_temperature@3x.png"
          title="设置温度"
          right-text="60℃"
          :icon-color="deviceThemeColorFn()"
          :has-sub-bottom-border="true"
          :has-arrow="true"
          :cell-style="topBorderRadius"
          :disabled="deviceDisableStatusComp"
          @dofCellClicked="pickTempreture"
        ></dof-cell>
        <dof-cell
          avatar-icon="http://dolphin-weex-dev.msmartlife.cn/cdn/images/business/other/calorifier_icon_eco@3x.png"
          title="节能模式"
          desc="一键45℃，热量不浪费"
          :icon-color="deviceThemeColorFn()"
          :has-sub-bottom-border="true"
          :has-arrow="false"
          :disabled="deviceDisableStatusComp"
          @dofCellClicked="cellClickedHandler"
        >
          <dof-switch
            slot="switch"
            :type="deviceThemeColorFn('switch')"
            :checked="switchValue"
            :disabled="deviceDisableStatusComp"
            @dof-change="handleSwitchChange({ type: '开关切换' })"
          ></dof-switch>
        </dof-cell>
        <dof-cell
          title="E+增容"
          desc="增容加热技术，提供更多热水"
          :icon-color="deviceThemeColorFn()"
          avatar-icon="http://dolphin-weex-dev.msmartlife.cn/cdn/images/business/other/calorifier_icon_e+@3x.png"
          :cell-style="bottomBorderRadius"
          :has-arrow="false"
          :disabled="deviceDisableStatusComp"
          @dofCellClicked="cellClickedHandler"
        >
          <dof-switch
            slot="switch"
            :type="deviceThemeColorFn('switch')"
            :checked="switchValue"
            :disabled="deviceDisableStatusComp"
            @dof-change="handleSwitchChange({ type: '开关切换' })"
          ></dof-switch>
        </dof-cell>
      </div>
    </scroller>
    <dof-minibar
      :style="{ position: 'fixed', top: 0 }"
      :title="deviceName"
      :right-button="moreIcon"
      :background-color="minibarTransitionBg"
      :has-bottom-border="minibarBottomLine"
      @dofMinibarRightButtonClicked="minibarRightButtonClick"
    >
    </dof-minibar>
    <dof-actionsheet
      ref="modelPicker"
      :show="modelPicker"
      title="选择加热模式"
      :button="buttonType"
      :titleStyle="modeTitleStyle"
      @close="modelPicker = false"
      @btnClick="cancelClicked"
    >
      <div
        class="mode-content-slot"
        slot="content"
      >
        <scroller class="model-scroller">
          <dof-cell
            v-for="(item, index) in modeList"
            :key="index"
            :title="item.title"
            :desc="item.desc"
            :icon-color="item.iconColor"
            :avatar-icon="item.icon"
            backgroundColor="#F9F9F9"
            :has-margin="true"
            @dofCellClicked="modelItemClicked(index)"
          >
          </dof-cell>
        </scroller>
      </div>
    </dof-actionsheet>
    <dof-modal
      title="设备已离线"
      content="请检查设备状态"
      :show="modalFlag"
      :single="single"
      @close="closeModal"
      @dofModalConfirmBtnClicked="dofModalConfirmBtnClicked"
    >
    </dof-modal>
    <dof-bottom-bar
      :tabGroups="tabTitles"
      @dofTabItemClicked="bottomBarOperateHandler"
    ></dof-bottom-bar>
  </div>
</template>

<script>
import {
  DofMinibar,
  DofBottomBar,
  DofActionsheet,
  DofCell,
  DofSwitch,
  DofModal,
  Utils
} from 'dolphin-weex-ui'
import IndicatorDashboard from './IndicatorDashboard'
import {
  MOCK_BOTTOM_BAR_DATA,
  ASSETS_DATA_HEATER as ASSETS_DATA
} from './type.js'
import NativeService from '@/service/nativeService.js'
const picker = weex.requireModule('picker')

export default {
  components: {
    DofMinibar,
    DofBottomBar,
    DofActionsheet,
    DofCell,
    DofSwitch,
    DofModal,
    IndicatorDashboard
  },
  data: () => ({
    deviceName: '电热水器',
    leftButton: './assets/image/header/icon_back_white@3x.png',
    moreIcon: './assets/image/header/icon_more_black@3x.png',
    deviceNumber: 20,
    deviceStatusCode: 1, //0:已离线 1: 已关机 2:保温中
    panelDescData: [
      {
        title: '热水量',
        value: '80%'
      },
      {
        title: '可洗浴时间',
        value: '≈ 30′'
      }
    ],
    baseConfig: ASSETS_DATA,
    switchValue: false,
    device: {
      power: 'off'
    },
    //Y方向上滚动的距离
    scrollDistanceY: 0,
    tabTitles: MOCK_BOTTOM_BAR_DATA,
    //modal
    single: false,
    modalFlag: false,
    modeList: [
      // {
      //   code: 2,
      //   title: '保温',
      //   desc: '保持当前温度',
      //   iconColor: '#ffaa10',
      //   icon: 'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_half@3x.png'
      // },
      {
        code: 3,
        title: '整胆加热',
        desc: '需要一定时间，提供大量热水',
        iconColor: '#ff8225',
        icon:
          'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_all@3x.png'
      },
      {
        code: 4,
        title: '抑菌加热',
        desc: '消毒',
        iconColor: '#ff6a4c',
        icon:
          'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_disinfect@3x.png'
      },
      {
        code: 5,
        title: '待机中',
        desc: '测试使用',
        iconColor: '#29c3ff',
        icon:
          'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_eco@3x.png'
      },
      {
        code: 0,
        title: '已离线',
        desc: '测试使用',
        iconColor: '#7c879b',
        icon:
          'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_on@3x.png'
      }
    ],
    buttonType: ['取消'],
    modelPicker: false,
    tempreture: '30', //温度
    tempretureIndex: 0 //温度索引
  }),
  created() {
    this.isIPhoneX = Utils.env.isIPhoneX()
  },
  mounted() {
    this.$set(this.device, 'power', this.device.power === 'on' ? 'off' : 'on')
  },
  computed: {
    //scroller底部设置
    fitBottomSize() {
      return this.isIPhoneX ? '256px' : '182px'
    },
    //minibar设置
    navHeight() {
      let result = '20'
      if (weex.config.env.statusBarHeight) {
        if (weex.config.env.platform === 'iOS') {
          // iOS使用pt为单位
          result = weex.config.env.statusBarHeight
        } else {
          // 安卓使用px为单位
          result = weex.config.env.statusBarHeight / weex.config.env.scale
        }
      }
      return (+result + 44) * 2
    },
    minibarTransitionBg() {
      let opacityValue =
        (1 / this.navHeight) * Math.min(this.scrollDistanceY, this.navHeight) <
        0.2
          ? 0
          : (1 / this.navHeight) *
            Math.min(this.scrollDistanceY, this.navHeight)
      return `rgba(255, 255, 255, ${opacityValue})`
    },
    minibarBottomLine() {
      let { scrollDistanceY } = this
      return scrollDistanceY > 20 ? true : false
    },

    topBorderRadius() {
      return {
        borderTopLeftRadius: '16px',
        borderTopRightRadius: '16px'
      }
    },
    bottomBorderRadius() {
      return {
        borderBottomLeftRadius: '16px',
        borderBottomRightRadius: '16px'
      }
    },
    modeTitleStyle() {
      return {
        textAlign: 'center',
        fontSize: '28px',
        color: '#8A8A8F',
        lineHeight: '48px'
      }
    },
    //自动计算处理设备禁用状态
    deviceDisableStatusComp() {
      let { deviceStatusCode } = this
      if (deviceStatusCode === 0 || deviceStatusCode === 1) {
        return true
      } else {
        return false
      }
    }
  },
  methods: {
    minibarRightButtonClick() {
      let nextPageUrl = 'meijv-template-page-more.js'
      this.$push(nextPageUrl)
      // this.$reload()
    },
    pageScrollHandler(e) {
      let {
        contentOffset: { y }
      } = e
      this.scrollDistanceY = Math.abs(y)
    },
    //列表点击
    cellClickedHandler() {
      this.$toast('cell clicked')
    },
    //开关切换
    handleSwitchChange(e) {
      this.switchValue = !this.switchValue
    },
    //自动计算处理主题颜色
    deviceThemeColorFn(type = 'hex') {
      let { deviceStatusCode } = this
      if (type === 'switch') {
        return this._fnThemeColorName(deviceStatusCode)
      } else {
        return this._fnThemeColorValue(deviceStatusCode)
      }
    },
    //计算颜色名称
    _fnThemeColorName(code) {
      if (code === 0 || code === 1) {
        return 'gray'
      } else if (code === 2) {
        return 'yellow'
      } else if (code === 3) {
        return 'orange'
      } else if (code === 4) {
        return 'tomato'
      } else if (code === 5) {
        return 'aqua'
      }
    },
    //计算颜色十六进制色值
    _fnThemeColorValue(code) {
      if (code === 0 || code === 1) {
        return '#7c879b'
      } else if (code === 2) {
        return '#ffaa10'
      } else if (code === 3) {
        return '#ff8225'
      } else if (code === 4) {
        return '#ff6a4c'
      } else if (code === 5) {
        return '#29c3ff'
      }
    },
    //bottomBar切换
    bottomBarOperateHandler(e) {
      let { target, index } = e

      //震动效果
      NativeService.hapticFeedback(1)

      if (target.title === 'power') {
        this.$set(
          this.device,
          'power',
          this.device.power === 'on' ? 'off' : 'on'
        )
      } else if (target.title === 'timing') {
        let timingUrl = 'meijv-template-page-timing.js'
        this.$push(timingUrl)
      } else if (target.title === 'modePicker') {
        this.modelPicker = true
        this.$nextTick(() => {
          this.$refs.modelPicker.open()
        })
      }
    },
    modelItemClicked(index) {
      this.modelPicker = false
      let item = this.modeList[index]
      this.deviceStatusCode = item.code
      if (item.code === 4) {
        this.deviceNumber = '80'
      } else if (item.code === 2) {
        this.deviceNumber = '36'
      } else if (item.code === 3) {
        this.deviceNumber = '46'
      } else if (item.code === 5) {
        this.deviceNumber = '26'
      }
    },
    cancelClicked() {
      this.modelPicker = false
    },
    //modal
    dofModalConfirmBtnClicked() {
      this.modalFlag = false
      setTimeout(() => {
        this.$set(this.device, 'power', 'off')
      }, 800)
    },
    closeModal() {},
    pickTempreture() {
      const items = [
        20,
        21,
        22,
        23,
        24,
        25,
        26,
        27,
        28,
        29,
        30,
        31,
        32,
        33,
        34,
        35,
        36,
        37,
        38,
        39
      ]
      var that = this
      var pickItem = [
        {
          index: that.tempretureIndex,
          item: items,
          label: '℃'
        }
      ]
      picker.pick(
        {
          items: pickItem,
          headTitle: '请选择温度', //取消和确定中间那标题
          cancelTxt: '取消', //取消按钮文字
          confirmTxt: '确定', //确定按钮文字,
          cancelTxtColor: '#000000', //取消颜色
          confirmTxtColor: '#000000', //标题颜色
          titleColor: '#000000', //标题颜色
          titleBgColor: '#E7EDEF' //标题栏颜色
        },
        event => {
          if (event.result == 'success') {
            var dataArr = event.data
              .replace('[', '')
              .replace(']', '')
              .split(',')
            that.tempretureIndex = dataArr[0]
            that.tempreture = items[dataArr[0]]
          }

          modal.toast({
            message: `您选择了第${that.tempretureIndex}个温度,温度为${that.tempreture}}`
          })
        }
      )
    }
  },
  watch: {
    'device.power': {
      handler(nv, ov) {
        if (nv === 'on') {
          this.deviceStatusCode = 2
        } else if (nv === 'off') {
          this.deviceStatusCode = 1
        }
      }
    },
    deviceStatusCode: {
      handler(nv, ov) {
        this.tabTitles.forEach((item, index) => {
          item.disabled = [0].includes(nv) ? true : false
          if (item.title === 'modePicker' || item.title === 'timing') {
            item.iconColor = this._fnThemeColorValue(nv)
            item.disabled = [0, 1].includes(nv) ? true : false
          }
          if (item.title === 'power') {
            item.iconColor = this._fnThemeColorValue(nv)
            nv === 1 ? (item.text = '开机') : (item.text = '关机')
            nv === 1
              ? (item.icon =
                  'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_on@3x.png')
              : (item.icon =
                  'http://dolphin-weex-dev.msmartlife.cn/cdn/images/sample/water_heater/calorifier_icon_on@3x.png')
          }
        })
        // this.$alert(this.tabTitles)
        if (nv === 0) {
          setTimeout(() => {
            this.single = true
            this.modalFlag = true
          }, 500)
        }
      },
      immediate: true
    }
  }
}
</script>

<style lang="scss" scoped>
@import 'src/css/dolphinweex';
.device {
  background-color: $background;
  &-scroll-view {
    // background-color: $aqua;
  }
  &-action-board {
    padding: 0 32px;
    margin: 40px 0;
  }
}

.mode-content-slot {
  justify-content: space-between;
  flex: 1;
  padding-left: 32px;
  padding-right: 32px;
}
.model-scroller {
  // height: 936px;
}
</style>
