<style lang="scss" scoped>
.noData {
  text-align: center;
  font-size: 14px;
  color: #333;
  height: 35px;
  line-height: 35px;
}
.move {
  cursor: move;
}
.noDrop {
  cursor: no-drop;
}
.pointer {
  cursor: pointer;
}
.body {
  font-size: 12px;
  color: #1f2d3d;
  cursor: pointer;
  .img {
    height: 22px;
    width: 22px;
  }
  .header {
    background: #f2f2f2;
    font-size: 14px;
    font-weight: bold;
    font-size: 14px;
    width: 100%;
    height: 30px;
    line-height: 30px;
  }
  .row {
    // height: 45px;
  }
  .row:hover {
    background: #f7fcf7;
  }
  .operator {
    display: inline-block;
    width: 12px;
    height: 13px;
  }
  .space {
    padding-left: 16px;
  }
  .tool {
    width: 7px;
    margin-right: 4px;
    height: auto;
    cursor: move;
  }
  .row {
    border-bottom: 1px solid #eeeeee;
    height: auto;
  }
  .text-overflow {
    // overflow: hidden;
    // text-overflow: ellipsis;
  }
  .is-hide {
    color: #b3b3b3;
  }
  .hash {
    color: #009919;
  }
  .tree-bg {
    background: #e1e1e173;
  }
}
</style>
<template>
  <div
    :class="{
      body: tableData.length !== 0,
      noData: tableData.length === 0,
    }"
    ref="$body"
  >
    <template v-for="(item, index) in tableData">
      <div
        :level="`${item.level}`"
        :key="`${item.level}_${item.id}`"
        :id="`${item.id}`"
      >
        <div class="row">
          <template v-for="(val, i) in tableHeader">
            <component
              :is="
                val.component ||
                (item.isChange == val.props ? item.component : false) ||
                'div'
              "
              :key="val.props"
              :class="{ cell: true, [`cell_${i}`]: true }"
              :style="{ 'padding-left': val.tree ? `${item.level * 5}px` : 0 }"
              v-bind="item.attr"
              v-on="item.listener"
            >
              <template v-if="val.select">
                <el-checkbox
                  v-bind="val.attr"
                  @change="handlerSelection(item, $event)"
                ></el-checkbox>
              </template>
              <template v-else-if="val.tree">
                <i
                  :class="{
                    'el-icon-caret-bottom': item.isShowIcon,
                    'el-icon-caret-right': !item.isShowIcon,
                  }"
                  v-if="iSShow(item.children)"
                  @click="onClick(item, $event)"
                ></i>
                <i v-else class="operator"></i>
                <img
                  :src="val.images || `@/assets/imgs/green.png`"
                  class="tool"
                  v-if="isSort"
                />
                {{ val.before ? val.before(item) : item[val.props] }}
              </template>
              <template v-else-if="val.actions"
                ><el-button
                  v-for="(btn, i) in val.actions"
                  :key="`${btn.label}_${i}`"
                  v-bind="btn.attr"
                  @click="btn.click(item, index, btn, $event)"
                  :disabled="btn.before ? btn.before(item, index) : false"
                  >{{ btn.label }}</el-button
                ></template
              >
              <template v-else>
                {{ val.before ? val.before(item) : item[val.props] }}</template
              >
            </component>
          </template>
        </div>
        <recursiveRow
          :data="item.children"
          v-show="iSShow(item.children) && item.isShowIcon"
          :tableHeader="tableHeader"
          :isSort="isSort"
          :class="{ 'tree-bg': item.children.length && item.isShowIcon }"
          :callback="callback"
          ref="recursiveRow"
        ></recursiveRow>
      </div>
    </template>
    <template v-if="tableData.length == 0"> 暂无数据 </template>
  </div>
</template>
<script lang="ts">
import "./style/index.scss";
import { Component, Vue, Prop, Watch, Ref } from "vue-property-decorator";
import Sortable from "sortablejs";
import * as _ from "lodash";
let $selectArr: any = [];
@Component({
  name: "recursiveRow",
})
export default class extends Vue {
  @Ref() recursiveRow!: any;
  @Ref() $body!: any;
  @Prop({
    type: Array,
    default() {
      return [];
    },
  })
  data?: [];
  @Prop({
    type: Boolean,
    default: false,
  })
  isSort?: false;
  @Prop({
    type: Function,
    default: () => {},
  })
  callback!: () => {};
  @Prop({
    type: Array,
    default: () => [],
  })
  tableHeader: any;
  private tableData: any = [];
  private sortable: any = {};
  private selectArr: any = [];
  mounted() {}
  private handlerSelection(row: any, bool: boolean) {
    if (bool) {
      $selectArr.push(row);
    } else {
      $selectArr = $selectArr.filter((rw: any) => rw.id !== row.id);
    }
    this.$emit("select-change", $selectArr);
  }
  private iSShow(children: Array<any>) {
    if (children && children.length) {
      return true;
    }
    return false;
  }
  //树形打开
  private onClick(...args: any) {
    const [item, evt] = args;
    evt.preventDefault();
    evt.stopPropagation();
    item.isShowIcon = !item.isShowIcon;
    this.$forceUpdate();
  }
  //全部打开
  private allOpen(expand: boolean) {
    this.handlerTableData(this.tableData, expand);
    this.recursiveRow &&
      this.recursiveRow.forEach((el: any) => {
        el.allOpen(expand);
      });
  }
  //阻止冒泡事件
  private handlerStop(evt: any) {
    evt.preventDefault();
    evt.stopPropagation();
  }
  private handlerTableData(arr: Array<any>, expand: boolean) {
    Array.isArray(arr) &&
      arr.forEach((item: any) => {
        this.$set(item, "isShowIcon", expand);
      });
  }
  //处理排序
  private handlerSort() {
    if (this.isSort === false) {
      this.sortable &&
        this.sortable.option &&
        this.sortable.option("disabled", !this.isSort);
      return;
    }
    const el = this.$body;
    this.sortable =
      el &&
      new Sortable(el, {
        // 结束拖拽
        onEnd: async function (this: any, evt: any = {}) {
          if (evt.oldIndex === evt.newIndex) return;
          [this.tableData[evt.oldIndex], this.tableData[evt.newIndex]] = [
            this.tableData[evt.newIndex],
            this.tableData[evt.oldIndex],
          ];
          //同层级的id
          let arr = this.tableData.map((item: any = {}) => item.id);
          //触发成功之后的回调
          await this.callback(arr);
          this.$message({
            type: "success",
            message: "拖拽排序成功",
          });
        }.bind(this),
        onMove: function (evt: any = {}) {
          if (evt.dragged.id === evt.related.id) return false;
        },
      });
  }
  @Watch("data", { deep: true, immediate: true })
  handlerData(newValue: any) {
    this.tableData = _.cloneDeep(newValue);
    this.handlerTableData(this.tableData, false);
    this.handlerSort();
  }
  @Watch("isSort")
  handlerIsSort(newValue: boolean) {
    this.handlerSort();
  }
}
</script>
