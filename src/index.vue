<template>
  <!--
    组件名称：联想输入框
    功能点：
      1.keyup或focus时触发联想；
      2.上下键加回车或鼠标选中联想结果；
      3.回车、选中、失去焦点均能向父组件返回输入值；
    props说明：
      1.reset用于清空输入框内容，使用时应该先赋值true，50ms后再赋值false
   -->
  <div class="think_input_wrapper">
    <input
      type="text"
      class="think_input"
      v-model="inputVal"
      :placeholder="placeholder"
      :style="styleJson"
      @keyup="handleKeyup"
      @blur="handleBlur"
      @focus="handleFocus"
    >
    <ul class="think_list" v-show="hasThinkResultFlag" :style="styleJson">
      <li
        class="think_result"
        v-for="(item, index) of thinkResult"
        :key="index"
        @click.stop="handleListClick(item)"
        :class="{active: index == focusIndex}"
        :title="item"
      >
        {{item}}
      </li>
    </ul>
    <div class="think_shade" v-show="hasThinkResultFlag" @click="handleShadeClick"></div>
  </div>
</template>

<script>
export default {
  name: "ThinkInputS",
  props: {
    // 默认文本
    placeholder: {
      type: String,
      required: false,
      default: '请输入...'
    },
    // 联想数据
    listData: {
      type: Array,
      required: false,
      default: function() {
        return  []
      }
    },
    // 联想框唯一名称(用于拼接事件名称)
    inputName: {
      type: String,
      required: false,
      default: 'myInput'
    },
    // 定制化样式
    styleJson: {
      type: Object,
      required: false,
      default: function() {
        return  {
          width: '185px'
        }
      }
    },
    // 清空联想框
    reset: {
      type: Boolean,
      required: false,
      default: false
    }
  },
  data() {
    return {
      inputVal: "",
      thinkResult: [],
      focusIndex: 0,
      hasThinkResultFlag: false,
      timer: null
    };
  },
  watch: {
    // 获取最新联想数据
    listData(newVal, oldVal) {
      this.thinkResult = newVal.slice()
    },
    // 清空联想框
    reset(newVal, oldVal) {
      if (newVal) {
        this.inputVal = ''
      }
    },
    // 联想数据变化时，显示下拉框
    thinkResult(newVal, oldVal) {
      if (newVal.length) {
        this.hasThinkResultFlag = true
      } else {
        this.hasThinkResultFlag = false
      }
    }
  },
  mounted() {
    // 拷贝联想数据
    this.thinkResult = this.listData.slice()
  },
  methods: {
    /**
     * 输入内容
     */
    handleKeyup(e) {
      if (this.inputVal == '') {
        this.hasThinkResultFlag = false
        this.exportValueAndEvent(this.inputVal, 'keyup')
      }
      e = e || window.event
      if (e.keyCode === 38) { // 向上
        if (this.focusIndex > 0) {
          this.focusIndex -= 1
        } else {
          this.focusIndex = 0
        }
      } else if (e.keyCode === 40) {  // 向下
        if (this.focusIndex < this.thinkResult.length - 1) {
          this.focusIndex += 1
        } else {
          this.focusIndex = this.thinkResult.length - 1
        }
      } else if (this.hasThinkResultFlag && e.keyCode === 13) { // 显示下拉框时，回车取值、返回值
        this.inputVal = this.thinkResult[this.focusIndex]
        this.hasThinkResultFlag = false
        this.exportValueAndEvent(this.inputVal, 'selectEnter')
      } else if (!this.hasThinkResultFlag && e.keyCode === 13) {  // 无下拉框时，回车仅返回值
        this.hasThinkResultFlag = false
        this.exportValueAndEvent(this.inputVal, 'enter')
      } else {
        // 输入时，节流处理，300ms内无新的输入才会输出 值
        if (this.timer) {
          clearTimeout(this.timer)
        }
        this.timer = setTimeout(() => {
          this.exportValueAndEvent(this.inputVal, 'keyup')
        }, 300)
      }
    },

    /**
     * 失去焦点
     * 由于blur早于click触发，导致无法鼠标点选，故对blur做延时触发，且鼠标点选时不再执行blur
     */
    handleBlur() {
      if (!this.hasThinkResultFlag) {
        this.exportValueAndEvent(this.inputVal, 'blur')
      } else {
        this.timer = setTimeout(() => {
          this.hasThinkResultFlag = false
          this.exportValueAndEvent(this.inputVal, 'blur')
        }, 300)
      }
    },

    /**
     * 获取焦点
     */
    handleFocus() {
      if (this.inputVal) {
        this.exportValueAndEvent(this.inputVal, 'focus')
      } else {
        this.hasThinkResultFlag = false
      }
    },

    /**
     * 鼠标点选
     */
    handleListClick(val) {
      clearTimeout(this.timer)
      this.inputVal = val
      this.hasThinkResultFlag = false
      this.exportValueAndEvent(this.inputVal, 'click')
    },

    /**
     * 输出 值(inputVal) 和 事件名(eventName)
     */
    exportValueAndEvent(inputVal, eventName) {
      this.$emit(`export${this.inputName}ValueAndEvent`, inputVal, eventName)
    },

    /**
     * 点击遮罩
     */
    handleShadeClick() {
      this.handleBlur()
    }
  }
};
</script>

<style lang="less" scoped>
.think_input_wrapper {
  display: inline-block;
  position: relative;
  .think_input {
    width: 185px;
    height: 23px;
    padding-left: 4px;
    border: 1px solid #d5dce2;
    user-select: none;
  }
  .think_list {
    position: absolute;
    top: 24px;
    right: 0;
    z-index: 10;
    width: 185px;
    background: #ffffff;
    border: 1px solid #d5dce2;
    list-style: none;
    padding: 0;
    margin: 0;
    .think_result {
      height: 20px;
      line-height: 20px;
      padding-left: 8px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      &:hover {
        cursor: pointer;
        background: #5183BC;
        color: #ffffff;
      }
      &.active {
        background: #5183BC;
        color: #ffffff;
      }
    }
  }
  .think_shade {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 1;
  }
}
</style>