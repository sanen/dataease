<template>
  <div v-show="menuShow" class="contextmenu" :style="{ top: menuTop + 'px', left: menuLeft + 'px' }">
    <ul @mouseup="handleMouseUp">
      <template v-if="curComponent">
        <template v-if="!curComponent.isLock">
          <li v-if="editFilter.includes(curComponent.type)" @click="edit"> {{ $t('panel.edit') }}</li>
          <li @click="copy"> {{ $t('panel.copy') }}</li>
          <li @click="paste"> {{ $t('panel.paste') }}</li>
          <li @click="cut"> {{ $t('panel.cut') }}</li>
          <li @click="deleteComponent"> {{ $t('panel.delete') }}</li>
          <!--          <li @click="lock"> {{ $t('panel.lock') }}</li>-->
          <li @click="topComponent"> {{ $t('panel.topComponent') }}</li>
          <li @click="bottomComponent"> {{ $t('panel.bottomComponent') }}</li>
          <li @click="upComponent"> {{ $t('panel.upComponent') }}</li>
          <li @click="downComponent"> {{ $t('panel.downComponent') }}</li>
        </template>
        <li v-else @click="unlock">解锁</li>
      </template>
      <li v-else @click="paste">粘贴</li>
    </ul>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import bus from '@/utils/bus'

export default {
  data() {
    return {
      copyData: null,
      editFilter: [
        'view',
        'v-text',
        'rect-shape',
        'custom'
      ]
    }
  },
  computed: mapState([
    'menuTop',
    'menuLeft',
    'menuShow',
    'curComponent',
    'componentData',
    'canvasStyleData'
  ]),
  methods: {
    edit() {
      // 编辑时临时保存 当前修改的画布
      this.$store.dispatch('panel/setComponentDataTemp', JSON.stringify(this.componentData))
      this.$store.dispatch('panel/setCanvasStyleDataTemp', JSON.stringify(this.canvasStyleData))
      if (this.curComponent.type === 'view') {
        this.$store.dispatch('chart/setViewId', null)
        this.$store.dispatch('chart/setViewId', this.curComponent.propValue.viewId)
        bus.$emit('PanelSwitchComponent', { name: 'ChartEdit', param: { 'id': this.curComponent.propValue.viewId }})
      }
      if (this.curComponent.type === 'custom') {
        bus.$emit('component-dialog-edit')
      }

      // 编辑样式组件

      if (this.curComponent.type === 'v-text' || this.curComponent.type === 'rect-shape') {
        bus.$emit('component-dialog-style')
      }
    },
    lock() {
      this.$store.commit('lock')
    },

    unlock() {
      this.$store.commit('unlock')
    },

    // 点击菜单时不取消当前组件的选中状态
    handleMouseUp() {
      this.$store.commit('setClickComponentStatus', true)
    },

    cut() {
      this.deleteCurCondition()
      this.$store.commit('cut')
    },

    copy() {
      this.$store.commit('copy')
    },

    paste() {
      this.$store.commit('paste', true)
      this.$store.commit('recordSnapshot')
    },

    deleteComponent() {
      this.deleteCurCondition()
      this.$store.commit('deleteComponent')
      this.$store.commit('recordSnapshot')
    },

    deleteCurCondition() {
      if (this.curComponent.type === 'custom') {
        this.$store.commit('removeViewFilter', this.curComponent.id)
        bus.$emit('delete-condition', { componentId: this.curComponent.id })
      }
    },

    upComponent() {
      this.$store.commit('upComponent')
      this.$store.commit('recordSnapshot')
    },

    downComponent() {
      this.$store.commit('downComponent')
      this.$store.commit('recordSnapshot')
    },

    topComponent() {
      this.$store.commit('topComponent')
      this.$store.commit('recordSnapshot')
    },

    bottomComponent() {
      this.$store.commit('bottomComponent')
      this.$store.commit('recordSnapshot')
    }
  }
}
</script>

<style lang="scss" scoped>
.contextmenu {
    position: absolute;
    z-index: 1000;

    ul {
        border: 1px solid #e4e7ed;
        border-radius: 4px;
        background-color: #fff;
        box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
        box-sizing: border-box;
        margin: 5px 0;
        padding: 6px 0;

        li {
            font-size: 14px;
            padding: 0 20px;
            position: relative;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            color: #606266;
            height: 34px;
            line-height: 34px;
            box-sizing: border-box;
            cursor: pointer;

            &:hover {
                background-color: #f5f7fa;
            }
        }
    }
}
</style>
