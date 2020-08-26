<template>
  <span style="display: block; width: 100%" :draggable="drag" @dragstart="handleDragStart">
    <a-tooltip
      :visible="visible"
      overlayClassName="u-text-tooltip"
      placement="bottomLeft"
      :getPopupContainer="getPopupContainer"
      :destroyTooltipOnHide="true"
      :align="align"
      @visibleChange="handleVisibleChange"
    >
      <template slot="title">
        <div class="tooltip-text">{{text}}</div>
      </template>
      <span
        ref="text"
        @mouseenter="handleMouseEnter"
        style="display: block;width: 100%"
        :style="`--count: ${maxLineCount}`"
        :class="className"
      >
        {{text}}
      </span>
    </a-tooltip>
  </span>
</template>

<script lang='ts'>
/*
 * 文本省略打点时自动添加tooltip
*/
import { Component, Prop, Vue } from 'vue-property-decorator'
import { Popover } from 'ant-design-vue'
const elementResizeDetectorMaker = require('element-resize-detector')
const erdUltraFast = elementResizeDetectorMaker({
  strategy: 'scroll'
})

@Component({
  components: {
    'a-popover': Popover
  }
})
export default class UTextTooltip extends Vue {
  @Prop() text!: string
  @Prop({
    type: Number,
    default: 1
  }) maxLineCount?: number

  @Prop({ default: false, type: Boolean }) drag!: boolean
  @Prop({ default: () => () => document.body }) getPopupContainer: Function

  visible = false
  tooltipShow = false
  targetNode: Element = null

  mouseOffsetX = 0

  get align () {
    const offset = [this.mouseOffsetX, -4]
    return {
      offset: offset
    }
  }

  get className () {
    return this.maxLineCount > 1 ? 'ellipsis-multiline' : 'ellipsis'
  }

  mounted () {
    this.targetNode = this.$refs.text as Element
    erdUltraFast.listenTo(this.targetNode, (element: HTMLElement) => {
      const offsetWidth = element.offsetWidth
      const offsetHeight = element.offsetHeight
      const scrollWidth = element.scrollWidth
      const scrollHeight = element.scrollHeight
      this.tooltipShow = (scrollWidth > offsetWidth) || (scrollHeight > offsetHeight)
    })
  }

  beforeDestroy () {
    erdUltraFast.uninstall(this.targetNode)
  }

  private handleVisibleChange (visible: boolean) {
    this.visible = this.tooltipShow ? visible : false
  }

  private handleDragStart (e: DragEvent) {
    this.$emit('dragstart', e)
  }

  private handleMouseEnter (e: MouseEvent) {
    this.mouseOffsetX = e.offsetX
  }
}
</script>

<style lang='less' scoped>
.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.ellipsis-multiline {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box !important;
  -webkit-line-clamp: var(--count);
  -webkit-box-orient: vertical;
}

.tooltip-text {
 max-width: 500px;
}
</style>

<style lang="less">
.u-text-tooltip {
  .ant-popover-arrow {
    display: none;
  }
  .ant-popover-inner {
    border: 1px solid #DEDEDE;
    box-shadow: 0 1px 2px 0 rgba(0,0,0,0.08);
    .ant-popover-inner-content {
      padding: 2px 8px;
      font-size: 13px;
    }
  }
}
</style>
