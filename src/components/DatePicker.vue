<template>
  <div class="datepicker">
    <div class="datepicker-header"></div>
    <div class="datepicker-content">
      <DatePickerItem
        v-for="(col, idx) in defaultProps.dateConfig"
        :key="idx"
        :item="col"
        :date="defaultProps.value"
        :max="defaultProps.max"
        :min="defaultProps.min"
        @onSelect="onSelectDate"
      />
    </div>
    <div class="datepicker-navbar">
      <div class="datepicker-navbar-btn">{{defaultProps.confirmText}}</div>
      <div class="datepicker-navbar-btn">{{defaultProps.cancelText}}</div>
    </div>
  </div>
</template>
<script>
import DatePickerItem from "./DatePickerItem.vue";
const defaultProps = {
  isPopup: true,
  isOpen: false,
  theme: "default",
  value: new Date(),
  min: new Date(1970, 0, 1),
  max: new Date(2050, 0, 1),
  showFooter: true,
  showHeader: true,
  showCaption: false,
  dateConfig: {
    year: {
      format: "YYYY",
      caption: "Year",
      step: 1,
      type: "Year"
    },
    month: {
      format: "M",
      caption: "Mon",
      step: 1,
      type: "Month"
    },
    date: {
      format: "D",
      caption: "Day",
      step: 1,
      type: "Date"
    }
  },
  headerFormat: "YYYY/MM/DD",
  confirmText: "完成",
  cancelText: "取消"
};
export default {
  components: {
    DatePickerItem
  },
  data() {
    return {
      confirmText: "OK",
      cancelText: "Cancel",
      defaultProps
    };
  },
  methods: {
    onSelectDate(date) {
      // eslint-disable-next-line no-console
      console.log(date);
      this.defaultProps.value = date;
    }
  }
};
</script>
<style lang="less" scoped>
.datepicker {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  z-index: 1;
  font-size: 16px;
  text-align: center;
  font-family: arial;
  box-sizing: content-box;
  user-select: none;

  .datepicker-header {
    padding: 0 0.5em;
    min-height: 2em;
    line-height: 2em;
    font-size: 1.125em;
  }
  .datepicker-content {
    display: flex;
    padding: 0.5em 0.25em;
  }
  .datepicker-navbar {
    padding: 0 0.5em 0.5em 0.5em;
    overflow: hidden;
  }
  .datepicker-navbar-btn {
    height: 2.5em;
    line-height: 2.5em;
    float: right;
    padding: 0 1em;
    cursor: pointer;
  }
}
</style>