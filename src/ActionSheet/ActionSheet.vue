<template>
  <Popup position="bottom" :maskClosable="maskClosable" v-model="localVisible">
    <div styleName="@action-sheet" v-show="localVisible">
      <div styleName="title" v-if="title || $slots.title">
        <slot name="title">
          {{ title }}
        </slot>
      </div>
      <div styleName="message" v-if="message || $slots.message">
        <slot name="message">
          {{ message }}
        </slot>
      </div>
      <div styleName="list">
        <div
          :styleName="[
            'item',
            !!item.danger && 'danger'
          ]"
          :style="{ color: item.color }"
          @click="handleItemClick(item, index)"
          v-for="(item, index) in localData"
          :key="index">
          <slot :item="item" :index="index">
            {{ item.label }}
          </slot>
        </div>
        <div styleName="item cancel" @click="hide" v-if="showCancel">
          <div styleName="mask" />
          <slot name="cancelText">
            {{ cancelText }}
          </slot>
        </div>
      </div>
    </div>
  </Popup>
</template>

<script>
import CSSModules from 'vue-css-modules'
import { normalizeData, isThenable } from '../_utils'
import { toggleVisibility } from '../_mixins'
import Popup from '../Popup/Popup.vue'

export default {
  name: 'ActionSheet',

  inject: ['ActionSheetStyles'],

  mixins: [
    toggleVisibility(false, true),
    CSSModules('ActionSheetStyles')
  ],

  props: {
    title: String,
    message: String,
    data: {
      type: Array,
      default: () => [],
      watch: true
    },
    showCancel: {
      type: Boolean,
      default: true
    },
    cancelText: {
      type: String,
      default: '取消'
    },
    maskClosable: {
      type: Boolean,
      default: true
    }
  },

  methods: {
    onReceiveData(data, transform) {
      transform(normalizeData(data))
    },
    handleItemClick(item, index) {
      this.$emit('item-click', item, index)
      if (item.onClick) {
        const result = item.onClick(item, index)
        if (isThenable(result)) {
          result.then(this.hide)
        } else {
          this.hide()
        }
      } else {
        this.hide()
      }
    }
  },

  components: { Popup }
}
</script>