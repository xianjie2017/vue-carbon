<template>
  <item-form :label="label" :icon="icon" :focus="focus" :no-empty="selectText ? true : false">
    <div class="vc-select" v-el:select>
      <div class="vc-select-text" @click="toggleSelect()" :class="{'placeholder': !selectText}">{{selectText || placeholder}}</div>
      <icon value="arrow_drop_down"  @click="toggleSelect()"></icon>
      <div class="vc-select-drop-down" :class="{'up': up}" v-show="focus">
        <ul class="vc-select-options">
          <li v-for="option in options">
            <a href="javascript:;" v-el:option @click="select(option)"
              class="vc-select-option" :class="{'selected': isSelect(option)}">
              <div class="vc-select-option-content">
                {{option.text || option}}
              </div>
              <icon value="check"></icon>
              <ripple :trigger="$els.option"></ripple>
            </a>
          </li>
        </ul>
      </div>
    </div>
  </item-form>
</template>

<script>
import itemForm from './itemForm'
import icon from '../icon/icon'
import ripple from '../ripple'
import * as domUtil from '../utils/domUtil'
export default {
  props: {
    label: {
      type: String,
      default: ''
    },
    icon: {
      type: String,
      default: ''
    },
    focus: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: ''
    },
    multi: {
      type: Boolean,
      default: false
    },
    value: {
      type: [Array, Object, Number, String]
    },
    options: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    return {
      up: false
    }
  },
  computed: {
    selectText () {
      let text = []
      this.options.forEach((item) => {
        if (this.isSelect(item)) {
          text.push(item.text || item)
        }
      })
      return text.join(',')
    }
  },
  attached () {
    this.addListener()
  },
  detached () {
    this.removeListener()
  },
  beforeDestroy () {
    this.removeListener()
  },
  methods: {
    isSelect (val) {
      return this.multi ? this.value && this.value.indexOf(val) !== -1 : this.value === val
    },
    toggleSelect () {
      this.focus = !this.focus
    },
    select (val) {
      if (this.multi) {
        if (!this.value) this.value = []
        const i = this.value.indexOf(val)
        if (i !== -1) {
          this.value.splice(i, 1)
        } else {
          this.value.push(val)
        }
      } else {
        this.value = val
        this.hideSelect()
      }
      this.$emit('input-change', this.value)
    },
    hideSelect () {
      this.focus = false
    },
    addListener () {
      this.windowListener = (e) => {
        if (!this.$els.select.contains(e.target)) {
          this.hideSelect()
        }
      }
      window.addEventListener('click', this.windowListener, false)
    },
    removeListener () {
      if (this.windowListener) {
        window.removeEventListener('click', this.windowListener, false)
        this.windowListener = null
      }
    }
  },
  watch: {
    focus (value) {
      if (value) {
        let stop = domUtil.getOffset(this.$els.select).top
        let wh = window.innerHeight
        this.up = wh - stop < 260
      }
    }
  },
  components: {
    'item-form': itemForm,
    icon,
    ripple
  }
}
</script>

<style lang="less">
@import "../utils/_vars.less";
@import "../utils/_mixins.less";
.vc-select{
  width: 100%;
  height: 36px;
  display: flex;
  position: relative;
  color: @color;
  align-items: center;
  .hairline(bottom, #d3d6db);
  &:after {
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
    -webkit-perspective: 1000;
    perspective: 1000;
    .translate3d();
    transition-duration: 200ms;
  }
  &.focus-state:after,
  &.not-empty-state:after,
  .focus-state &:after,
  .not-empty-state &:after {
    background: @red;
    .transform(scaleY(2)) !important;
  }
  > .icon-arrow_drop_down {
    .flex-shrink(0)
  }
}

.vc-select-text{
  flex: 1;
  font-size: 16px;
  height: 36px;
  line-height: 36px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  word-wrap: break-word;
  &.placeholder{
    color: @body_color;
  }
}

.vc-select-drop-down{
  position: absolute;
  width: 100%;
  top: 36px;
  left: 0;
  .depth(2);
  z-index: 20;
  background-color: #FFF;
  &.up{
    top: auto;
    bottom: 36px;
  }
}
.vc-select-options{
  margin: 0;
  list-style: none;
  padding: 0;
  max-height: 216px;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
}
.vc-select-option{
  display: flex;
  align-items: center;
  font-size: 16px;
  padding: 6px 12px;
  min-height: 36px;
  color: @color;
  position: relative;
  .vc-ripple-ink{
    color: rgba(0, 0, 0, .1);
  }
  &.selected{
    color: @red;
    > .icon{
      display: inline-block;
      vertical-align: middle;
    }
  }
  > .icon {
    display: none;
  }
}

.vc-select-option-content{
  flex: 1;
}
</style>
