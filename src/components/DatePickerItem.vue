/* eslint-disable no-console */
/* eslint-disable no-console */
<template>
  <div class="datepicker-col-1">
    <div
      class="datepicker-viewport"
      @touchstart="handleContentTouch"
      @touchmove="handleContentTouch"
      @touchend="handleContentTouch"
      @mousedown="handleContentMouseDown"
      @mousemove="handleContentMouseMove"
      @mouseup="handleContentMouseUp"
    >
      <div class="datepicker-wheel">
        <ul class="datepicker-scroll" :style="Y">
          <li v-for="(date, idx) in dates" :key="idx">{{renderDatePicker(date)}}</li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script>
import * as TimeUtil from "../time";
const DATE_HEIGHT = 40;
const DATE_LENGTH = 10;
const MIDDLE_INDEX = Math.floor(DATE_LENGTH / 2);
const MIDDLE_Y = -DATE_HEIGHT * MIDDLE_INDEX;
export default {
  props: {
    item: {
      type: Object,
      default: () => ({})
    },
    date: {
      type: Date,
      default: new Date()
    },
    min: {
      type: Date,
      default: 0
    },
    max: {
      type: Date,
      default: 0
    }
  },
  data() {
    return {
      animating: false,
      touchY: 0,
      translateY: MIDDLE_Y,
      currentIndex: MIDDLE_INDEX,
      moveDateCount: 0,
      dates: []
    };
  },
  watch: {
    date: function(newDate) {
      // eslint-disable-next-line no-console
      console.log(newDate);
      if (newDate.getTime() === this.date.getTime()) {
        return;
      }
      this._iniDates(newDate);
      this.currentIndex = MIDDLE_INDEX;
    }
  },
  mounted() {
    this._iniDates(this.date);
  },
  computed: {
    marginTop() {
      return (this.currentIndex - MIDDLE_INDEX) * DATE_HEIGHT;
    },
    Y() {
      return {
        transform: `translateY(${this.translateY}px)`,
        marginTop: this.marginTop
      };
    }
  },
  methods: {
    _iniDates(date) {
      // eslint-disable-next-line no-console
      // console.log(date);
      const typeName = this.item.type;
      const dates = Array(...Array(DATE_LENGTH)).map((value, index) =>
        TimeUtil[`next${typeName}`](
          date,
          (index - MIDDLE_INDEX) * this.item.step
        )
      );
      this.dates = dates;
    },
    renderDatePicker(date) {
      // const className =
      //     (date < this.props.min || date > this.props.max) ?
      //     'disabled' : '';

      let formatDate;

      formatDate = TimeUtil.convertDate(date, this.item.format);
      // eslint-disable-next-line no-console
      // console.log(date, formatDate);
      return formatDate;
    },
    handleContentTouch() {
      event.preventDefault();
      if (this.animating) return;
      if (event.type === "touchstart") {
        this.handleStart(event);
      } else if (event.type === "touchmove") {
        this.handleMove(event);
      } else if (event.type === "touchend") {
        this.handleEnd(event);
      }
    },
    handleEnd(event) {
      const touchY = event.pageY || event.changedTouches[0].pageY;
      const dir = touchY - this.touchY;
      const direction = dir > 0 ? -1 : 1;
      this._moveToNext(direction);
    },
    handleStart() {
      this.touchY =
        event.targetTouches && event.targetTouches[0]
          ? event.targetTouches[0].pageY
          : event.pageY;

      this.translateY = this.translateY;
      this.moveDateCount = 0;
    },

    handleMove(event) {
      const touchY =
        event.targetTouches && event.targetTouches[0]
          ? event.targetTouches[0].pageY
          : event.pageY;

      const dir = touchY - this.touchY;
      const translateY = this.translateY + dir;
      const direction = dir > 0 ? -1 : 1;

      // 日期最小值，最大值限制
      const date = this.dates[MIDDLE_INDEX];
      const { max, min } = this;
      if (date.getTime() < min.getTime() || date.getTime() > max.getTime()) {
        return;
      }

      // 检测是否更新日期列表
      if (this._checkIsUpdateDates(direction, translateY)) {
        this.moveDateCount =
          direction > 0 ? this.moveDateCount + 1 : this.moveDateCount - 1;
        this._updateDates(direction);
      }

      this.translateY = translateY;
    },
    _updateDates(direction) {
      const typeName = this.item.type;
      let { dates } = this;
      if (direction === 1) {
        this.currentIndex++;

        dates = [
          ...dates.slice(1),
          TimeUtil[`next${typeName}`](dates[dates.length - 1], this.item.step)
        ];
      } else {
        this.currentIndex--;

        this.dates = [
          TimeUtil[`next${typeName}`](dates[0], -this.item.step),
          ...dates.slice(0, dates.length - 1)
        ];
      }
    },
    _checkIsUpdateDates(direction, translateY) {
      return direction === 1
        ? this.currentIndex * DATE_HEIGHT + DATE_HEIGHT / 2 < -translateY
        : this.currentIndex * DATE_HEIGHT - DATE_HEIGHT / 2 > -translateY;
    },
    handleContentMouseUp() {},
    handleContentMouseDown(event) {
      if (this.animating) return;
      this.handleStart(event);
    },
    handleContentMouseMove() {
      if (this.animating) return;
      this.handleMove(event);
    },
    _moveTo(currentIndex) {
      this.animating = true;

      // addPrefixCss(this.refs.scroll, { transition: 'transform .2s ease-out' });

      this.translateY = -currentIndex * DATE_HEIGHT;

      // NOTE: There is no transitionend, setTimeout is used instead.
      this._moveToTimer = setTimeout(() => {
        this.animating = false;
        this.$emit("onSelect", this.dates[MIDDLE_INDEX]);
        this._clearTransition(this.$refs.scroll);
      }, 200);
    },
    _moveToNext(direction) {
      const date = this.dates[MIDDLE_INDEX];
      const { max, min } = this;
      if (
        direction === -1 &&
        date.getTime() < min.getTime() &&
        this.moveDateCount
      ) {
        this._updateDates(1);
      } else if (
        direction === 1 &&
        date.getTime() > max.getTime() &&
        this.moveDateCount
      ) {
        this._updateDates(-1);
      }

      this._moveTo(this.currentIndex);
    }
  }
};
</script>
<style lang="less" scoped>
.datepicker-col-1 {
  flex: 1;
  margin: 0 0.25em;
}

.datepicker-viewport {
  height: 200px;
  position: relative;
  overflow: hidden;
  &::after {
    content: "";
    position: absolute;
    z-index: 2;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    pointer-events: none;
  }
}

.datepicker-wheel {
  position: absolute;
  height: 40px;
  top: 50%;
  margin-top: -20px;
  width: 100%;
  border-top: 1px solid #4eccc4;
  border-bottom: 1px solid #4eccc4;
}

.datepicker-scroll {
  list-style-type: none;
  padding: 0;
  & > li {
    height: 40px;
    line-height: 40px;
    font-size: 1.375em;
    cursor: pointer;
  }
}
</style>