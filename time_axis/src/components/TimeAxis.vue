<template>
  <div class="axis-time-cnpnt">
    <div class="top-date">
      {{ getMiddleTime("date") }}
      {{ getMiddleTime("time") }}
    </div>
    <div class="timeline">
      <div
        class="container"
        @mousedown="startDrag"
        :style="{ width: visibleRangeWidth }"
      >
        <div class="visible-range" :style="{ width: visibleRangeWidth }">
          <div
            class="total-range"
            :style="{
              left: rangeLeftLocation,
              width: totalRangeWidth,
              backgroundColor: rangeParams.globalColor,
            }"
          >
            <div
              class="range-color"
              v-for="item of rangeList"
              :key="item.id"
              :style="{
                left: `${item.left}px`,
                width: `${item.width}px`,
                backgroundColor: item.timeRangeColor,
              }"
            ></div>
            <div class="scale-marks">
              <span
                class="scale-mark-line"
                v-for="(mark, i) of scaleMarks"
                :key="mark"
                :style="{
                  left: mark - scaleMarkWidth / 2 - 1 + 'px',
                  width: `${scaleMarkWidth}px`,
                  height: `${scaleMarkHeight}px`,
                }"
              >
                <span
                  :class="i % 24 != 0 ? 'time-value-i' : 'time-value-date'"
                  >{{ showDate(i) }}</span
                >
              </span>
            </div>
          </div>
          <div class="middle-axis-line">
            <div class="date-box time-box">
              <div class="time-border">
                <!-- <el-input-number @change="updateTimeAxis()" class="custom-input-number" size="small" v-model="dateTime.hour" controls-position="right" :min="0" :max="23"></el-input-number> -->
                <!-- <el-input-number @change="updateTimeAxis()" class="custom-input-number" size="small" v-model="dateTime.min" controls-position="right" :min="0" :max="59"></el-input-number> -->
                <!-- <el-input-number @change="updateTimeAxis()" class="custom-input-number" size="small" v-model="dateTime.sec" controls-position="right" :min="0" :max="59"></el-input-number> -->
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="time-picker">
      <!-- <el-input-number @change="updateTimeAxis()"
                       class="custom-input-number"
                       size="small"
                       v-model="dateTime.hour"
                       controls-position="right"
                       :min="0"
                       :max="23"></el-input-number>
      <el-input-number @change="updateTimeAxis()"
                       class="custom-input-number"
                       size="small"
                       v-model="dateTime.min"
                       controls-position="right"
                       :min="0"
                       :max="59"></el-input-number>
      <el-input-number @change="updateTimeAxis()"
                       class="custom-input-number"
                       size="small"
                       v-model="dateTime.sec"
                       controls-position="right"
                       :min="0"
                       :max="59"></el-input-number> -->
    </div>
    <!-- <el-time-picker class="time-picker" @change="handleTimePickerChange" v-model="timePickerValue" placeholder="任意时间点">
    </el-time-picker> -->
  </div>
</template>

<script>
function addLeadingZero(number) {
  if (number < 10) {
    return "0" + number;
  } else {
    return number.toString();
  }
}
// import { addLeadingZero } from "@/utils/common";
export default {
  props: {
    isActive: {
      type: Boolean,
      default: false,
    },
    // 默认日期
    value: {
      type: Array,
      default: function () {
        return [2024, 2, 1];
      },
    },
  },
  data() {
    return {
      timePickerValue: [],
      // 可视区宽度
      visibleRange: (8 * 60) / 2,
      // 刻度值列表
      scaleMarks: [0],
      // 可视区1px 代表一分钟
      minPerPx: 1,
      // 总共时长/h
      totalHours: 72,
      // 一小时多少分钟
      minutes: 60,
      // 刻度线宽度
      scaleMarkWidth: 1.5,
      // 刻度线高度
      scaleMarkHeight: 5,
      dragging: false,
      startX: 0,
      rangeLeft: 0,
      // 鼠标移动多少px算一分钟
      percent: 1,
      displayHours: 24,

      // 一天多少小时
      hoursOneDay: 24,
      // 时间戳列表
      dateArr: [0, 0, 0, 0],
      seconds: 60,
      milliseconds: 1000,
      rangeParams: {
        timeRangeColor: "#0077fa",
        startTime: 0,
        endTime: 0,
        globalColor: "#e4e4e5",
      },
      dateTime: {
        year: 2023,
        month: 8,
        day: 23,
        hour: 14,
        min: 59,
        sec: 59,
        time: 0,
        dateTime: "",
      },
      oldLeft: 0,
      colorLeft: "0px",
      colorWidth: "20px",
      dialogVisible: false,
      newParams: {
        timeRangeColor: "#0077fa",
        startTime: 0,
        endTime: 0,
        globalColor: "#e4e4e5",
      },
      dom: null,
      middleTime: 0,
      rangeList: [
        {
          startTime: 1692868244000,
          endTime: 1692871844000,
          globalColor: "#e4e4e5",
          timeRangeColor: "#0077fa",
        },
        // { startTime: 1687233600000, endTime: 1687437000000, globalColor: '#e4e4e5', timeRangeColor: '#3DCF66' },
        // { startTime: 1687534200000, endTime: 1687552200000, globalColor: '#e4e4e5', timeRangeColor: '#3DCF66' },
        // { startTime: 1687793400000, endTime: 1688070600000, globalColor: '#e4e4e5', timeRangeColor: '#0077fa' },
        // { startTime: 1688088600000, endTime: 1688128200000, globalColor: '#e4e4e5', timeRangeColor: '#3DCF66' },
      ],
      timer: null,
    };
  },

  methods: {
    handleTimePickerChange() {},
    getRangeNum(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    setRangeList() {
      // 获取昨天的日期
      const yesterday = new Date();
      this.rangeList = [];
      let num = this.getRangeNum(20, 30);
      let time = yesterday.getTime() - 0 * 60 * 60 * 1000;
      let endTime = time;
      let startTime = endTime - 30 * 24 * 60 * 60 * 1000;
      let step = 3 * 60 * 60 * 1000;
      // for (let i = 0; i < num; i++) {
      //   time = time - Math.ceil(Math.random() * step)
      //   this.rangeList.push({ endTime: time, timeRangeColor: '#0077fa', cameraCode: '1975' })
      //   time = time - Math.ceil(Math.random() * step)
      //   this.rangeList[i].startTime = time
      // }
      this.rangeList.push({
        endTime,
        startTime,
        timeRangeColor: "#0077fa",
        cameraCode: "1975",
      });
    },
    setDateTime(year = 2023, mon = 8, day = 24, hour = 1, min = 1, sec = 0) {
      this.dateTime.year = Number(year);
      this.dateTime.mon = Number(mon);
      this.dateTime.day = Number(day);
      this.dateTime.hour = Number(hour);
      this.dateTime.min = Number(min);
      this.dateTime.sec = Number(sec);
      this.dateTime.dateTime = new Date(year, mon - 1, day, hour, min, sec);
    },
    initDateTime() {
      let date = new Date();
      let year = date.getFullYear();
      let mon = date.getMonth() + 1;
      let day = date.getDate();
      let hour = date.getHours();
      let min = date.getMinutes();
      let sec = date.getSeconds();
      this.setDateTime(year, mon, day, hour, min, sec);
    },
    updateTimeAxis(isHandChange) {
      let year = this.dateTime.year;
      let mon = this.dateTime.mon - 1;
      let day = this.dateTime.day;
      let hour = this.dateTime.hour;
      let min = this.dateTime.min;
      let sec = this.dateTime.sec;
      this.dateTime.dateTime = new Date(year, mon, day, hour, min, sec);
      if (isHandChange) {
        this.$emit("select", [
          this.dateTime.year,
          this.dateTime.mon,
          this.dateTime.day,
        ]);
      }
      this.initData();
      this.sendDateTimeToVideo();
    },
    initData() {
      this.scaleMarks = [];
      let j = 0;
      for (let i = 0; i < this.totalHours; i++) {
        this.scaleMarks.push((i * this.minutes) / this.minPerPx);
        j = i;
      }
      this.displayHours =
        document
          .getElementsByClassName("axis-time-cnpnt")[0]
          .getBoundingClientRect()["width"] /
        this.minPerPx /
        this.minutes;
      this.scaleMarks.push(((j + 1) * this.minutes) / this.minPerPx);
      this.visibleRange = (this.displayHours * this.minutes) / this.minPerPx;
      // this.rangeLeft = this.hoursOneDay * this.minutes / this.minPerPx * (-1)
      const currentMs = this.dateTime.dateTime.getTime();
      const msOneDay =
        this.hoursOneDay * this.minutes * this.seconds * this.milliseconds;
      // currentMs=Math.floor(currentMs/msOneDay)*(msOneDay)
      this.dateArr = [];
      for (let i = -1; i < this.totalHours / this.hoursOneDay; i++) {
        this.dateArr.push(
          currentMs +
            i *
              this.hoursOneDay *
              this.minutes *
              this.seconds *
              this.milliseconds
        );
      }
      this.rangeParams.startTime = this.dateTime.dateTime;
      this.rangeParams.endTime = new Date(
        this.rangeParams.startTime.getTime() +
          this.displayHours * this.minutes * this.seconds * this.milliseconds
      );
      let currTime =
        (this.rangeParams.startTime.getHours() - this.displayHours / 2) *
          this.minutes +
        this.rangeParams.startTime.getMinutes();
      this.middleTime = currTime;
      this.rangeLeft = (currTime / this.minPerPx + 24 * 60) * -1;
      // for (let i = 0; i < this.rangeList.length; i++) {
      //   this.rangeList[i]['startTime']=new Date(this.rangeList[i]['startTime'])
      //   this.rangeList[i]['endTime']=new Date(this.rangeList[i]['endTime'])
      //   ;

      // }
      this.updateRangeList();
    },
    startDrag(event) {
      if (this.dragging) {
        return;
      }
      event.preventDefault();
      event.stopPropagation();
      this.dragging = true;
      this.startX = event.clientX;
      this.oldLeft = this.rangeLeft;
      document.addEventListener("mousemove", this.handleStartDrag);
      document.addEventListener("mouseup", this.stopDrag);
    },
    handleMouseDown(min) {
      this.dragging = true;
      this.updateRangeLeft(this.rangeLeft + min);
      let timer = setInterval(() => {
        this.updateRangeLeft(this.rangeLeft + min);
      }, 100);
      const removeEvent = () => {
        clearInterval(timer);
        this.dragging = false;
        document.removeEventListener("mouseup", removeEvent);
      };
      document.addEventListener("mouseup", removeEvent);
    },
    // 处理中间拖拽
    handleStartDrag(event) {
      if (!this.dragging) return;
      event.stopPropagation();
      // 鼠标位移
      let distance = (event.clientX - this.startX) / this.percent;
      const newLeft = this.oldLeft + distance;
      this.updateRangeLeft(newLeft, event);
      this.updateRangeList();
      this.getMiddleTime();
    },
    // 停止拖拽
    stopDrag() {
      this.dragging = false;
      this.sendDateTimeToVideo(true);
      document.removeEventListener("mousemove", this.handleStartDrag);
      document.removeEventListener("mouseup", this.stopDrag);
      this.$emit("select", [
        this.dateTime.year,
        this.dateTime.mon,
        this.dateTime.day,
      ]);
      console.log([this.dateTime]);
    },
    getMiddleTime(key) {
      let middleTime =
        (this.rangeLeft * -1 * this.minPerPx * 2 +
          this.displayHours * this.minutes) /
        2;
      let hours = Math.floor(middleTime / this.minutes);
      let mins = Math.ceil(middleTime % this.minutes);
      if (mins >= this.minutes) {
        hours += 1;
        mins = 0;
      }
      let i =
        Math.ceil(
          (this.rangeLeft * -1 +
            ((this.displayHours / 2) * this.minutes) / this.minPerPx +
            1) /
            this.minutes /
            this.hoursOneDay
        ) - 1;
      let date = new Date(this.dateArr[i]);
      let year = date.getFullYear();
      let month = addLeadingZero(date.getMonth() + 1);
      let day = date.getDate();
      let hour = addLeadingZero(hours % 24);
      let min = addLeadingZero(mins);
      this.setDateTime(year, month, day, hour, min, 0);
      let time = { hours, mins };
      let timeString = `${hour}:${min}`;
      this.middleTime = `${year}-${month}-${day} ${hour}:${min}`;
      let obj = {
        date: `${year} 年 ${month} 月 ${day}日`,
        time: `${hour} : ${min}`,
        obj: { year, month, day, hour, min },
        dateTime: `${year}-${month}-${day} ${hour}:${min}`,
        none: "",
      };
      return obj[key];
    },
    setRangeParams() {
      if (!this.rangeParams) {
        return;
      }
      this.newParams.startTime = this.rangeParams.startTime;
      this.newParams.endTime = this.rangeParams.endTime;
      this.newParams.globalColor = this.rangeParams.globalColor;
      this.newParams.timeRangeColor = this.rangeParams.timeRangeColor;
      this.dialogVisible = true;
    },
    handleParams(result) {
      let res = {};
      this.rangeParams = result;
      res.startTime = result.startTime.getTime();
      res.endTime = result.endTime.getTime();
      res.globalColor = result.globalColor;
      res.timeRangeColor = result.timeRangeColor;
      // );
      this.updateRangeList(res);
    },
    getDate(ms) {
      var date = new Date(ms);
      var year = date.getFullYear();
      var month = date.getMonth() + 1; // 月份从0开始，因此需要加1
      var day = date.getDate();
      if (isNaN(year) || isNaN(month) || isNaN(day)) {
        return "--月--日";
      }
      return `${month}月${day}日`;
    },

    getLastDayOfMonth(month) {
      const lastDay = new Date(
        this.dateTime.year,
        this.dateTime.month - 1,
        0
      ).getDate();
      return lastDay;
    },
    showDate(i) {
      let index = Math.floor(i / 24);
      return i % 24 == 0 ? this.getDate(this.dateArr[index]) : i % 24;
      // return '2014-23-12'
    },
    updateRangeLeft(newLeft, event) {
      if (newLeft >= 0) {
        this.rangeLeft =
          ((this.hoursOneDay * this.minutes) / this.minPerPx) * -1;
        this.oldLeft = this.rangeLeft;
        this.startX = event.clientX;
        // 时间戳列表减上一天的时间戳
        let ms =
          this.hoursOneDay * this.minutes * this.seconds * this.milliseconds;
        this.updateDateArr(ms * -1);
      } else if (
        newLeft <=
        (((this.hoursOneDay * 3 - this.displayHours) * this.minutes) /
          this.minPerPx) *
          -1
      ) {
        // 时间戳列表加上一天的时间戳
        this.oldLeft = this.rangeLeft;
        this.startX = event.clientX;
        this.rangeLeft =
          (((this.hoursOneDay * 2 - this.displayHours) * this.minutes) /
            this.minPerPx) *
          -1;
        let ms =
          this.hoursOneDay * this.minutes * this.seconds * this.milliseconds;
        this.updateDateArr(ms);
      } else {
        this.rangeLeft = newLeft;
      }
    },
    updateDateArr(ms) {
      for (let i = 0; i < this.dateArr.length; i++) {
        this.dateArr[i] = this.dateArr[i] + ms;
      }
    },
    updateRangeList(res) {
      // res.startTime=res.startTime.getTime()
      // res.endTime=res.endTime.getTime()
      if (res) {
        this.rangeList.push(res);
      }
      for (let i = 0; i < this.rangeList.length; i++) {
        let startTime = this.rangeList[i].startTime;
        let firstTime = new Date(this.dateArr[0]).setHours(0, 0, 0, 0);
        let left =
          (startTime - firstTime) /
          this.milliseconds /
          this.seconds /
          this.minPerPx;
        this.rangeList[i]["left"] = left;
        let endTime = this.rangeList[i].endTime;
        let lastTime = this.dateArr[this.dateArr.length - 1];
        let width = 0;
        // if(startTime>lastTime || endTime<startTime){
        //   return '0px'
        // }
        width =
          (endTime - startTime) /
          this.milliseconds /
          this.seconds /
          this.minPerPx;
        this.rangeList[i]["width"] = width;
      }
    },
    handleDialogClose() {
      this.dialogVisible = false;
    },
    confirm() {
      let start = this.newParams.startTime.getTime();
      let end = this.newParams.endTime.getTime();
      if (start > end) {
        this.$message.warning("时间范围有误");
        return;
      }
      this.handleDialogClose();
      this.handleParams(this.newParams);
    },
    sendDateTimeToVideo() {
      let time = this.dateTime.dateTime.getTime();
      let obj = {};
      this.timer && clearTimeout(this.timer);
      for (const item of this.rangeList) {
        if (item.startTime < time && time < item.endTime) {
          this.timer = setTimeout(() => {
            item.middleTime = time;
            this.$emit("listenDateTime", item);
          }, 500);
          break;
        }
      }
    },
  },
  mounted() {
    this.initDateTime();
    this.setRangeList();
    this.updateTimeAxis();
    this.sendDateTimeToVideo();
  },
  computed: {
    visibleRangeWidth() {
      return `${this.visibleRange}px`;
    },
    totalRangeWidth() {
      return `${(this.totalHours * this.minutes) / this.minPerPx}px`;
    },
    rangeLeftLocation() {
      return `${this.rangeLeft}px`;
    },
  },
  watch: {
    value() {
      let year = this.value[0];
      let mon = this.value[1];
      let day = this.value[2];
      let hour = this.dateTime.hour;
      let min = this.dateTime.min;
      let sec = this.dateTime.sec;
      this.setDateTime(year, mon, day, hour, min, sec);
      this.updateTimeAxis();
    },
  },
};
</script>
<style lang="scss" scoped>
* {
  user-select: none;
}
.axis-time-cnpnt {
  position: relative;
  left: 0;
  z-index: 99;
  width: 100%;
  min-height: 154px;
  height: calc(100% - 2rem);
  max-height: 180px;
  background-color: #fff;
  overflow: hidden;
  border: 1px solid gray;
  .top-date {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    font-size: 12px;
    font-family: Microsoft YaHei;
    color: #0077fa;
  }
  .timeline {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 100%;
    height: 100%;
    background-color: transparent;
    box-sizing: border-box;
    color: black;
    .container {
      height: 70%;
      position: absolute;
      top: 3.5rem;
      left: 50%;
      transform: translateX(-50%);
      z-index: 100;
      cursor: grab;
      .visible-range {
        height: 70%;
        position: absolute;
        left: 50%;
        top: -18px;
        transform: translateX(-50%);

        .total-range {
          position: absolute;
          top: 40%;
          transform: translateY(-50%);
          height: 5px;
          color: black;
          box-sizing: border-box;
          .range-color {
            position: absolute;
            top: 0;
            left: 0;
            width: 0;
            height: 5px;
          }
          .scale-marks {
            position: absolute;
            width: 100%;
            height: 1rem;
            top: -20px;
            .scale-mark-line {
              display: inline-block;
              position: absolute;
              background-color: #b5b8bf;
              .time-value-i,
              .time-value-date {
                position: absolute;
                left: 50%;
                transform: translate(-50%, -50%);
                font-size: 12px;
                color: #00000099;
                bottom: 4px;
              }
              .time-value-date {
                text-align: center;
                width: 6rem;
              }
            }
          }
        }
        .middle-axis-line {
          display: inline-block;
          box-sizing: border-box;
          height: 4.3125rem;
          width: 4px;
          background-color: #ff3434;
          position: absolute;
          left: 50%;
          top: 40%;
          transform: translate(-50%, -50%);
          border: solid 1px #f5f5f5;
          .middle-axis-time,
          .middle-axis-date {
            position: absolute;
            left: 50%;
            top: calc(100% + 5px);
            transform: translateX(-50%);
            text-align: center;
            color: #ff3434;
            font-size: 18px;
            /* background-color: #3DCF66; */
            border-radius: 10px;
            white-space: nowrap;
            width: 14rem;
          }
          .middle-axis-time {
            font-weight: 400;
          }
          .left-icon {
            float: left;
            cursor: pointer;
          }

          .right-icon {
            float: right;
            cursor: pointer;
          }
          .middle-axis-date {
            top: calc(100% + 2.5rem);
            font-size: 12px;
            /* font-family: Inter; */
            line-height: 9px;
            color: #00000099;
          }
          .axis-date {
            position: absolute;
            top: 100%;
            left: 50%;
            transform: translateX(-50%);
            width: auto;
            white-space: nowrap;
          }
          .date-box {
            position: relative;
            padding: 1px;
            width: max-content;
            z-index: 10;
          }
          .time-border {
            display: inline-block;
            border-radius: 4px;
            border: 1px solid #e8e9eb;
            display: flex;
            // gap: 10px;
            // background-color: orange;
          }
          .date-box .el-input-number {
            width: 100px;
            display: inline-block;
            padding: 1px;
            border-radius: 0;
            /* background-color: white; */
          }
          .time-box {
            position: absolute;
            left: 50%;
            top: calc(50% + 2rem);
            transform: translate(-50%, -50%);
          }

          ::v-deep.custom-input-number .el-input__inner {
            border-radius: 0 !important; /* Remove border-radius */
            -webkit-border-radius: 0px;
            border: none;
          }
        }
      }
    }
  }
  .time-picker {
    position: absolute;
    bottom: 30px;
    left: 50%;
    transform: translateX(-50%);
  }
}
</style>
