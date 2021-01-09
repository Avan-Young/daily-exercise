<template>
  <div class="date-picker" v-click-outside>
    <!-- input区域 -->
    <div class="picker-input">
      <span class="input-prefix">
        <i class="iconfont icon-date" />
      </span>
      <input type="text" :value="chooseDate" readonly />
    </div>
    <!-- 日历区域，这里如果用的是v-show，会有个小bug，与父元素的自定义命令触发的机制相关 -->
    <div class="picker-panel" v-if="showPanel">
      <!-- 小三角 -->
      <div class="picker-arrow" />
      <!-- 正式区域 -->
      <div class="picker-body">
        <!-- 控制区 -->
        <div class="picker-header">
          <span
            class="picker-btn iconfont icon-prev-year"
            @click="onChangeYear('prev')"
          />
          <span
            class="picker-btn iconfont icon-prev-month"
            @click="onChangeMonth('prev')"
          />
          <span class="picker-date">
            {{ this.showDate.year }}年{{ this.showDate.month }}月
          </span>
          <span
            class="picker-btn iconfont icon-next-month"
            @click="onChangeMonth('next')"
          />
          <span
            class="picker-btn iconfont icon-next-year"
            @click="onChangeYear('next')"
          />
        </div>
        <!-- 周、日期显示区 -->
        <div class="picker-content">
          <div class="picker-weeks">
            <div
              v-for="week in ['日', '一', '二', '三', '四', '五', '六']"
              :key="week"
            >
              {{ week }}
            </div>
          </div>
          <div class="picker-days">
            <div
              v-for="date in showDay"
              :key="date.getTime()"
              :class="{
                'other-month': !isCur(date).month,
                'is-select': isCur(date).select,
                'is-today': isCur(date).today,
              }"
              @click.prevent="onChooseDate(date)"
            >
              {{ date.getDate() }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  directives: {
    // 只点击自身才显示日历，反之隐藏的指令
    "click-outside": {
      bind(el, binding, vnode) {
        // 获取指令当前元素所在实例，一会需要使用它内部的data和methods做逻辑处理
        const vm = vnode.context;

        document.onclick = function (e) {
          // 判断当前点击元素是否包含在内的子元素（已优化，子元素用了absolute定位，不占据父元素宽高）
          const dom = e.target;
          const isElSon = el.contains(dom);
          console.log(el,dom);
          if (isElSon && !vm.showPanel) {
            // 点击是自身内子元素且原隐藏 -> 触发显示
            vm.changePanel(true);
          } else if (!isElSon && vm.showPanel) {
            // 点击不是自身内子元素且已显示 -> 触发隐藏
            vm.changePanel(false);
          }
        };
      },
    },
  },
  model: {
    prop: "date",
    event: "choose-date",
  },
  props: {
    date: {
      type: Date,
      default: () => new Date(),
    },
  },
  computed: {
    // 传递过来的日期不是理想的展示格式，转换一下
    chooseDate() {
      const { year, month, day } = this.formatDate(this.date);
      return `${year}-${month}-${day}`;
    },
    showDay() {
      // 根据显示的年月来渲染日历
      const { year, month } = this.showDate;
      // 得到本月第一天，然后得到它是周几
      const firstDay = new Date(year, month - 1, 1);
      const week = firstDay.getDay();
      // 得到日历的第一天：本月第一天 - 周几*一天的毫秒数 得到的正是多出来上个月的那几天（这方法妙妙妙！）
      const second4Day = 24 * 60 * 60 * 1000;
      const startDay = firstDay - week * second4Day;

      let arr = [];
      for (let i = 0; i < 42; i++) {
        arr.push(new Date(startDay + i * second4Day));
      }
      return arr;
    },
  },
  data() {
    return {
      showPanel: false, // 是否显示日历
      showDate: {
        // 控制显示日历的年月
        year: 0,
        month: 0,
        day: 0,
      },
    };
  },
  methods: {
    // 获取显示的日期
    getShowDate(date) {
      const { year, month, day } = this.formatDate(date);
      this.showDate = {
        year,
        month,
        day,
      };
    },
    /**
     * 传入一个Date对象，进行日期格式化，返回一个对象，包含：
     * year，month，day，hour，minutes，second
     */
    formatDate(date) {
      const year = date.getFullYear(); // 获取年 yyyy
      const month = date.getMonth() + 1; // 获取月 0-11
      const day = date.getDate(); // 获取天 0-31
      const hour = date.getHours(); // 获取小时 0-23
      const minutes = date.getMinutes(); // 获取分钟 0-59
      const second = date.getSeconds(); // 获取秒 0-59

      return {
        year,
        month,
        day,
        hour,
        minutes,
        second,
      };
    },
    /** 显示隐藏日历 */
    changePanel(flag) {
      this.showPanel = flag;
    },
    /**
     * 用于判断是否当前选择日期、当前月日期、今天
     * @param (Date) date
     */
    isCur(date) {
      // 选择的日期，转为日期对象
      const chooseDate = new Date(this.chooseDate);

      const { year: showYear, month: showMonth } = this.showDate;
      const {
        year: chooseYear,
        month: chooseMonth,
        day: chooseDay,
      } = this.formatDate(chooseDate);
      const { year: curYear, month: curMonth, day: curDay } = this.formatDate(
        new Date()
      );

      const { year, month, day } = this.formatDate(date);

      return {
        month: year === showYear && month === showMonth,
        select:
          year === chooseYear && month === chooseMonth && day === chooseDay,
        today: year === curYear && month === curMonth && day === curDay,
      };
    },
    /**
     * 点击某一天，更改显示选择的日期，由于计算属性的chooseDate是根据
     * 父组件传递过来的date影响的，所以需要使用$emit来传递事件让父组件修改
     */
    onChooseDate(date) {
      this.$emit("choose-date", date);
      this.changePanel(false);
      this.getShowDate(date);
    },
    /** 更改月份 */
    onChangeMonth(type) {
      const { year, month, day } = this.showDate;
      const moveMonth = type === "prev" ? -1 : 1;
      const showDate = new Date(year, month - 1, day);
      showDate.setMonth(month - 1 + moveMonth);
      const { year: showYear, month: showMonth } = this.formatDate(showDate);
      this.showDate.year = showYear;
      this.showDate.month = showMonth;
    },
    /** 更改年份 */
    onChangeYear(type) {
      const moveYear = type === "prev" ? -1 : 1;
      this.showDate.year += moveYear;
    },
  },
  created() {
    this.getShowDate(this.date);
  },
};
</script>

<style scoped>
@import url("./assets/font.css");
.date-picker {
  user-select: none;
  /* 这里设置inline-block是配合自定义的命令 */
  display: inline-block;
}

.picker-input {
  position: relative;
}

.picker-input input {
  height: 40px;
  line-height: 40px;
  padding: 0 30px;
  background-color: #fff;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  outline: none;
  cursor: pointer;
}

.picker-input .input-prefix {
  position: absolute;
  left: 5px;
  width: 25px;
  height: 100%;
  line-height: 40px;
  text-align: center;
  color: #c0c4cc;
}

.picker-panel {
  position: absolute;
  width: 322px;
  height: 329px;
  margin-top: 5px;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

.picker-panel .picker-arrow {
  position: absolute;
  top: -12px;
  left: 30px;
  width: 0;
  height: 0;
  border: 6px solid transparent;
  border-bottom-color: #ebeef5;
}

.picker-panel .picker-arrow::after {
  position: absolute;
  left: -6px;
  top: 1px;
  content: "";
  display: block;
  width: 0;
  height: 0;
  border: 6px solid transparent;
  border-bottom-color: #fff;
  border-top-width: 0;
}

.picker-panel .picker-header {
  display: flex;
  align-items: center;
  justify-content: center;
  padding-top: 15px;
  padding-bottom: 10px;
}
.picker-panel .picker-btn {
  margin-right: 5px;
  margin-left: 5px;
  font-size: 12px;
  color: #303133;
  cursor: pointer;
}

.picker-panel .picker-date {
  margin-left: 60px;
  margin-right: 60px;
  font-size: 14px;
  /* user-select: none; */
}

.picker-panel .picker-content {
  padding: 0 10px 10px 10px;
  color: #606266;
  /* user-select: none; */
}

.picker-panel .picker-weeks {
  display: flex;
  justify-content: space-around;
  height: 40px;
  line-height: 40px;
  border-bottom: 1px solid #ebeef5;
}

.picker-panel .picker-days {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.picker-panel .picker-days div {
  width: 30px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  margin: 4px 6px;
  font-size: 12px;
  cursor: pointer;
}

.picker-panel .picker-days div:hover {
  color: #409eff;
}

.picker-panel .picker-days div.is-today {
  color: #409eff;
  font-weight: 700;
}

.picker-panel .picker-days div.is-select {
  border-radius: 50%;
  background-color: #409eff;
  color: #fff;
}

.picker-panel .picker-days div.other-month {
  color: #c0c4cc;
}
</style>