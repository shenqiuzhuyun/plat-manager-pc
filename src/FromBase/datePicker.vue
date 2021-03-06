<template>
  <div class="fe-single-data-picker">
    <el-form-item
      :label="endTitle | capitalize"
      :class="{'is-required': isRequired}"
    >
      <el-date-picker
        v-model="defaultValue"
        :placeholder="placeholder"
        :disabled="disabled"
        :value-format="format"
        ref="input"
        :type="dateType"
        :picker-options="dataOptions"
        class="fe-single-data-picker-input-content"
      >
      </el-date-picker>
      <!--<div-->
      <!--class="el-form-item__error error_input"-->
      <!--v-if="inForm && $v.defaultValue.$error"-->
      <!--&gt;{{errorMessage}}-->
      <!--</div>-->
    </el-form-item>
  </div>
</template>
<script>
  import validateMixin from './mixin/validateMixin'
  import baseMixin from './mixin/baseMixin'
  import moment from 'moment'
  import _ from 'lodash'

  export default {
    name: 'fe-single-date-picker',
    mixins: [validateMixin, baseMixin],
    props: {
      format: {
        type: String,
        default: 'yyyy-MM-dd'
      },
      value: {
        type: String | Array
      },
      limit: {
        type: Number
      },
      quickCard: {
        type: Object
      }
    },
    data() {
      this.dataOptions = {
        disabledDate: this.setDateRange,
        firstDayOfWeek: 1
      }
      return {
        defaultValue: ''
      }
    },
    computed: {
      dateType() {
        if (this.format.length < 12) {
          return 'date'
        } else {
          return 'datetime'
        }
      }
    },
    methods: {
      setDateRange(newDate) {
        let maxValue = (this.validate && this.validate.maxValue) || ''
        let minValue = (this.validate && this.validate.minValue) || ''
        if (this.quickCard) {
          const nowMax = moment()
          const nowMin = moment()
          const years = this.quickCard.years
          const months = this.quickCard.months
          const days = this.quickCard.days
          let beforeFlag = false
          let afterFlag = false
          if (years && !_.isEmpty(years)) {
            const before = years.before
            const after = years.after
            if (before !== undefined) {
              beforeFlag = true
              nowMin.subtract(before, 'years')
            }
            if (after !== undefined) {
              afterFlag = true
              nowMax.add(after, 'years')
            }
            /* nowMin.subtract(before, 'years')
             nowMax.add(after, 'years') */
          }
          if (months && !_.isEmpty(months)) {
            const before = months.before
            const after = months.after
            if (before !== undefined) {
              beforeFlag = true
              nowMin.subtract(before, 'months')
            }
            if (after !== undefined) {
              afterFlag = true
              nowMax.add(after, 'months')
            }
          }
          if (days && !_.isEmpty(days)) {
            const before = days.before
            const after = days.after
            if (before !== undefined) {
              beforeFlag = true
              nowMin.subtract(before, 'days')
            }
            if (after !== undefined) {
              afterFlag = true
              nowMax.add(after, 'days')
            }
          }
          if (beforeFlag) {
            minValue = nowMin
          }
          if (afterFlag) {
            maxValue = nowMax
          }
        }
        if (maxValue && minValue) {
          const max = moment(maxValue)
          const min = moment(minValue)
          return !((max > moment(newDate)) && moment(newDate) > min)
        } else if (maxValue) {
          const max = moment(maxValue)
          return !(max > moment(newDate))
        } else if (maxValue) {
          const min = moment(newDate)
          return !(moment(newDate) > min)
        } else {
          if (this.limit === 1) {
            return newDate.getTime() < (Date.now() - 86400000)
          } else {
            return false
          }
        }
      }
    },
    watch: {
      value: {
        handler(val) {
          if (process.env.PROJECT === 'bi' || process.env.PROJECT === 'app') {
            if (val) {
              this.defaultValue = val
            } else {
              this.defaultValue = moment().format('YYYY-MM-DD')
              this.$emit('input', this.defaultValue)
            }
          } else {
            this.defaultValue = val
          }
        },
        deep: true,
        immediate: true
      },
      defaultValue(val) {
        this.$emit('input', val)
      }
    }
  }
</script>
<style rel="stylesheet/scss" lang="scss">
  .fe-single-data-picker {
    width: unset;
    position: relative;
    margin-top: 5px;
    margin-bottom: 5px;
    .el-form-item {
      margin-bottom: 0px !important;
      &.s-required {
        &::before {
          content: "*";
          color: #f56c6c;
          margin-right: 4px;
          position: absolute;
          bottom: 5px;
          left: 0;
        }
      }
    }
    label {
      font-size: 12px;
      width: 100px;
      line-height: 30px;
      display: inline-block;
    }
    .fe-single-data-picker-input-content {
      width: 178px;
    }

  }
</style>
