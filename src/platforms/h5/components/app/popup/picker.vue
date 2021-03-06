<template>
  <uni-picker @touchmove.prevent>
    <transition
      name="uni-fade">
      <div
        v-show="visible"
        class="uni-mask"
        @click="_cancel" />
    </transition>
    <div
      :class="{'uni-picker_toggle':visible}"
      class="uni-picker">
      <div
        class="uni-picker-header"
        @click.stop>
        <div
          class="uni-picker-action uni-picker-action-cancel"
          @click="_cancel">取消</div>
        <div
          class="uni-picker-action uni-picker-action-confirm"
          @click="_change">确定</div>
      </div>
      <v-uni-picker-view
        :value.sync="valueArray"
        class="uni-picker-content">
        <v-uni-picker-view-column
          v-for="(range,index0) in rangeArray"
          :key="index0">
          <div
            v-for="(item,index) in range"
            :key="index"
            class="uni-picker-item">{{ typeof item==='object'?item[rangeKey]||'':item }}{{ units[index0]||'' }}</div>
        </v-uni-picker-view-column>
      </v-uni-picker-view>
      <!-- 第二种时间单位展示方式-暂时不用这种 -->
      <!-- <div v-if="units.length" class="uni-picker-units">
        <div v-for="(item,index) in units" :key="index">{{item}}</div>
      </div> -->
    </div>
  </uni-picker>
</template>
<script>
const mode = {
  SELECTOR: 'selector',
  MULTISELECTOR: 'multiSelector',
  TIME: 'time',
  DATE: 'date',
  REGION: 'region'
}
const fields = {
  YEAR: 'year',
  MONTH: 'month',
  DAY: 'day'
}
export default {
  name: 'Picker',
  props: {
    pageId: {
      type: Number,
      default: 0
    },
    range: {
      type: Array,
      default () {
        return []
      }
    },
    rangeKey: {
      type: String,
      default: ''
    },
    value: {
      type: [Number, String, Array],
      default: 0
    },
    mode: {
      type: String,
      default: mode.SELECTOR
    },
    fields: {
      type: String,
      default: fields.DAY
    },
    start: {
      type: String,
      default () {
        if (this.mode === mode.TIME) {
          return '00:00'
        }
        if (this.mode === mode.DATE) {
          let year = (new Date()).getFullYear() - 150
          switch (this.fields) {
            case fields.YEAR:
              return year
            case fields.MONTH:
              return year + '-01'
            case fields.DAY:
              return year + '-01-01'
          }
        }
        return ''
      }
    },
    end: {
      type: String,
      default () {
        if (this.mode === mode.TIME) {
          return '23:59'
        }
        if (this.mode === mode.DATE) {
          let year = (new Date()).getFullYear() + 150
          switch (this.fields) {
            case fields.YEAR:
              return year
            case fields.MONTH:
              return year + '-12'
            case fields.DAY:
              return year + '-12-31'
          }
        }
        return ''
      }
    },
    disabled: {
      type: [Boolean, String],
      default: false
    },
    customItem: {
      type: String,
      default: ''
    },
    visible: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      timeArray: [],
      dateArray: [],
      valueArray: []
    }
  },
  computed: {
    rangeArray () {
      var val = this.range
      switch (this.mode) {
        case mode.SELECTOR:
          return [val]
        case mode.MULTISELECTOR:
          return val
        case mode.TIME:
          return this.timeArray
        case mode.DATE: {
          let dateArray = this.dateArray
          switch (this.fields) {
            case fields.YEAR:
              return [dateArray[0]]
            case fields.MONTH:
              return [dateArray[0], dateArray[1]]
            case fields.DAY:
              return [dateArray[0], dateArray[1], dateArray[2]]
          }
        }
      }
    },
    startArray () {
      var splitStr = this.mode === mode.DATE ? '-' : ':'
      return this.start.split(splitStr).map((val, i) => (this.mode === mode.DATE ? this.dateArray : this.timeArray)[i].indexOf(val))
    },
    endArray () {
      var splitStr = this.mode === mode.DATE ? '-' : ':'
      return this.end.split(splitStr).map((val, i) => (this.mode === mode.DATE ? this.dateArray : this.timeArray)[i].indexOf(val))
    },
    units () {
      switch (this.mode) {
        case mode.DATE:
          return ['年', '月', '日']
        case mode.TIME:
          return ['时', '分']
        default:
          return []
      }
    }
  },
  watch: {
    valueArray (val) {
      if (this.mode === mode.TIME || this.mode === mode.DATE) {
        let getValue = this.mode === mode.TIME ? this._getTimeValue : this._getDateValue
        let valueArray = this.valueArray
        let startArray = this.startArray
        let endArray = this.endArray
        if (this.mode === mode.DATE) {
          const dateArray = this.dateArray
          let max = dateArray[2].length
          let day = dateArray[2][valueArray[2]]
          let realDay = new Date(`${dateArray[0][valueArray[0]]}/${dateArray[1][valueArray[1]]}/${day}`).getDate()
          day = Number(day)
          if (realDay < day) {
            valueArray[2] -= realDay + max - day
          }
        }
        if (getValue(valueArray) < getValue(startArray)) {
          this._cloneArray(valueArray, startArray)
        } else if (getValue(valueArray) > getValue(endArray)) {
          this._cloneArray(valueArray, endArray)
        }
      }
    }
  },
  created () {
    this._createTime()
    this._createDate()
    this._setValue(this.value)
    this.$watch('value', this._setValue)
    this.$watch('mode', this._setValue)
  },
  methods: {
    _createTime () {
      var hours = []
      var minutes = []
      hours.splice(0, hours.length)
      for (let i = 0; i < 24; i++) {
        hours.push((i < 10 ? '0' : '') + i)
      }
      minutes.splice(0, minutes.length)
      for (let i = 0; i < 60; i++) {
        minutes.push((i < 10 ? '0' : '') + i)
      }
      this.timeArray.push(hours, minutes)
    },
    _createDate () {
      var years = []
      var year = (new Date()).getFullYear()
      for (let i = year - 150, end = year + 150; i <= end; i++) {
        years.push(String(i))
      }
      var months = []
      for (let i = 1; i <= 12; i++) {
        months.push((i < 10 ? '0' : '') + i)
      }
      var days = []
      for (let i = 1; i <= 31; i++) {
        days.push((i < 10 ? '0' : '') + i)
      }
      this.dateArray.push(years, months, days)
    },
    _getTimeValue (val) {
      return val[0] * 60 + val[1]
    },
    _getDateValue (val) {
      return val[0] * 366 + (val[1] || 0 * 31) + (val[2] || 0)
    },
    /**
     * 将右侧数组值同步到左侧（交集部分）
     */
    _cloneArray (val1, val2) {
      for (let i = 0; i < val1.length && i < val2.length; i++) {
        val1[i] = val2[i]
      }
    },
    _setValue () {
      var val = this.value
      var valueArray
      switch (this.mode) {
        case mode.SELECTOR:
          valueArray = [val]
          break
        case mode.MULTISELECTOR:
          valueArray = [...val]
          break
        case mode.TIME:
          valueArray = val.split(':').map((val, i) => this.timeArray[i].indexOf(val))
          break
        case mode.DATE:
          valueArray = val.split('-').map((val, i) => this.dateArray[i].indexOf(val))
          break
      }
      this.valueArray = valueArray
    },
    _getValue () {
      var val = this.valueArray
      switch (this.mode) {
        case mode.SELECTOR:
          return val[0]
        case mode.MULTISELECTOR:
          return val.map(val => val)
        case mode.TIME:
          return this.valueArray.map((val, i) => this.timeArray[i][val]).join(':')
        case mode.DATE:
          return this.valueArray.map((val, i) => this.dateArray[i][val]).join('-')
      }
    },
    _change () {
      this.$emit('close')
      // 触发 View 层 change 事件
      UniServiceJSBridge.publishHandler(this.pageId + '-picker-change', {
        value: this._getValue()
      }, this.pageId)
    },
    // 暂时取消 columnchange 事件
    // _columnchange (event) {
    //   // 触发 View 层 columnchange 事件
    //   UniServiceJSBridge.publishHandler(this.pageId + '-picker-columnchange', {
    //     column: 0,
    //     value: this._getValue()
    //   }, this.pageId)
    // },
    _cancel () {
      // 通知父组件修改 visible
      this.$emit('close')
      // 触发 View 层 cancel 事件
      UniServiceJSBridge.publishHandler(this.pageId + '-picker-cancel', {}, this.pageId)
    }
  }
}
</script>
<style>
uni-picker {
  display: block;
  box-sizing: border-box;
}

uni-picker .uni-picker {
  position: fixed;
  left: 0;
  bottom: 0;
  transform: translate(0, 100%);
  backface-visibility: hidden;
  z-index: 999;
  width: 100%;
  background-color: #efeff4;
  transition: transform 0.3s;
}

uni-picker .uni-picker.uni-picker_toggle {
  transform: translate(0, 0);
}

uni-picker .uni-picker * {
  box-sizing: border-box;
}

uni-picker .uni-picker {
  position: fixed;
  left: 0;
  bottom: 0;
  transform: translate(0, 100%);
  backface-visibility: hidden;
  z-index: 999;
  width: 100%;
  background-color: #efeff4;
  transition: transform 0.3s;
  transition: transform 0.3s;
}

uni-picker .uni-picker-content {
  position: relative;
  display: block;
  width: 100%;
  height: 238px;
  background-color: white;
}
uni-picker .uni-picker-item {
  padding: 0;
  height: 34px;
  line-height: 34px;
  text-align: center;
  color: #000;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
uni-picker .uni-picker-header {
  display: block;
  position: relative;
  text-align: center;
  width: 100%;
  height: 45px;
  background-color: #fff;
}
uni-picker .uni-picker-header:after {
  content: "";
  position: absolute;
  left: 0;
  bottom: 0;
  right: 0;
  height: 1px;
  clear: both;
  border-bottom: 1px solid #e5e5e5;
  color: #e5e5e5;
  transform-origin: 0 100%;
  transform: scaleY(0.5);
}
uni-picker .uni-picker-action {
  display: block;
  max-width: 50%;
  top: 0;
  height: 100%;
  box-sizing: border-box;
  padding: 0 14px;
  font-size: 17px;
  line-height: 45px;
  overflow: hidden;
}
uni-picker .uni-picker-action.uni-picker-action-cancel {
  float: left;
  color: #888;
}
uni-picker .uni-picker-action.uni-picker-action-confirm {
  float: right;
  color: #007aff;
}
/* .uni-picker {
  position: relative;
}
.uni-picker-units {
  position: absolute;
  display: flex;
  width: 100%;
  line-height: 16px;
  font-size: 14px;
  top: 50%;
  margin-top: 22.5px;
  transform: translateY(-50%);
  overflow: hidden;
  color: #666666;
  pointer-events: none;
}
.uni-picker-units > div {
  flex: 1;
  text-align: center;
  transform: translateX(2em);
} */
</style>
