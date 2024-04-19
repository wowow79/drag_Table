<template>
    <div class="tableCard_mainDiv">
        <div class="tableCard_mainDiv_top">
            <div class="tableCard_mainDiv_top_title">{{title}}</div>
            <div class="tableCard_mainDiv_top_rightBtn">
                <slot name="btnSlot"></slot>
                <div>
                    <Icon type="md-refresh" @click="refreash" style="marginRight: 1vh;" />
                    <Icon type="md-settings" @click="showDrag" />
                </div>
            </div>
        </div>
        <div class="dragDiv" v-show="showDragDiv">
            <div class="dragDiv_top">
                <Checkbox v-model="checkAll" @on-change="check_all">列展示/排序</Checkbox>
                <span class="dragDiv_top_reset" @click="resetColumns">重置</span>
            </div>
            <Divider />
            <draggable v-model="columnsDrag" @end="onEnd" :key="dragKey">
                <transition-group>
                    <div class="dragDiv_item" v-for="(item, index) in columnsDrag" :key="item.key + 'drag'">
                        <Icon type="ios-list" size="20"/>
                        <Checkbox v-model="item._checked" @on-change="checkItem($event, item, index)">{{item.title}}</Checkbox>
                    </div>
                </transition-group>
            </draggable>
        </div>
        <Table
            :columns="insideColumns"
            :data="insideTableData"
            stripe
            :height="height"
            @on-selection-change="selectionChange">
            <slot name="action" slot="action"></slot>
        </Table>
        <div class="pageDiv">
            <Page
                class="pageDiv_page"
                :current="current"
                :total="total"
                :page-size="pageSize"
                show-sizer
                show-total
                @on-change="changePage"
                @on-page-size-change="changeSize"/>
        </div>
    </div>
</template>

<script>
    export default {
        name: '',
        props: {
            title: {
                type: String,
                default: () => {
                    return ''
                }
            },
            value: {
                type: Array,
                default () {
                    return []
                }
            },
            columns: {
                type: Array,
                default: () => {
                    return []
                }
            },
            height: {
                type: [Number, String]
            },
            current: {
                type: Number,
                default: () => {
                    return 1
                }
            },
            total: {
                type: Number,
                default: () => {
                    return 0
                }
            },
            pageSize: {
                type: Number,
                default: () => {
                    return 10
                }
            }
        },
        components: {

        },
        data () {
            return {
                showDragDiv: false,
                columnsDrag: [],
                columnsUse: [],
                checkAll: false,
                dragKey: 93,

                insideTableData: [],
                insideColumns: [],
                columnsAll: []
            }
        },
        watch: {
            value: {
                immediate: true,
                deep: true,
                handler () {
                    this.handleTableData()
                }
            },
            columns (val) {
                this.handleColumns(val)
            }
        },
        created () {
        },
        mounted () {
            this.handleColumns(this.columns)
            this.handleTableData()
        },
        methods: {
            refreash () {
                this.$emit('refreash')
            },
            showDrag () {
                this.showDragDiv = !this.showDragDiv
            },
            check_all (e) {
                this.columnsDrag.forEach(el => {
                    el._checked = e
                })
                this.onEnd()
            },
            resetColumns () {
                this.columnsDrag = JSON.parse(JSON.stringify(this.columns))
                this.checkAll = true
                this.columnsDrag.forEach(el => {
                    el._checked = true
                })
                this.dragKey++
                this.handleColumns(this.columns)
            },
            onEnd () {
                // 获取_checked为true的列
                var checkedColumns = this.columnsDrag.filter(el => el._checked)
                this.checkAll = checkedColumns.length === this.columnsDrag.length
                // 用于赋值表格的新列 从columnsAll完整列获取
                var newCol = []
                checkedColumns.forEach(el => {
                    this.columnsAll.forEach(elin => {
                        if (elin.key === el.key) {
                            newCol.push(elin)
                        }
                    })
                })
                this.insideColumns = newCol
            },
            checkItem (e, item, index) {
                // 改来改去没用了
                this.onEnd()
            },
            changePage (e) {
                this.$emit('changePage', e)
            },
            changeSize (e) {
                this.$emit('changeSize', e)
            },
            selectionChange (selection) {
                this.$emit('selectionChange', selection)
            },
            handleColumns (columns) {
                // 用于表格的列
                this.insideColumns = columns.map((item, index) => {
                    let res = item
                    if (res.needRender === true) res = this.surportHandle(res)
                    return res
                })

                // 保存一份完整的列
                this.columnsAll = columns.map((item, index) => {
                    let res = item
                    if (res.needRender === true) res = this.surportHandle(res)
                    return res
                })

                // 用于拖拽的数组 增加_checked属性
                this.columnsDrag = JSON.parse(JSON.stringify(this.columns))
                this.columnsDrag.forEach(el => {
                    el._checked = true
                })
                this.checkAll = true
            },
            surportHandle (item) {
                let insideBtns = []
                let btns = item.button ? [].concat(insideBtns, item.button) : insideBtns
                item.render = (h, params) => {
                    params.tableData = this.value
                    return h('div', btns.map(item => item(h, params, this)))
                }
                return item
            },
            handleTableData () {
                this.insideTableData = this.value.map((item, index) => {
                    let res = item
                    res.initRowIndex = index
                    return res
                })
            }
        }
    }
</script>

<style lang="less" scoped>
.tableCard_mainDiv {
    height: fit-content;
    background: #fff;
    padding: 1vh;
    border: 0.1vh solid #f0f0f0;
    border-radius: 1vh;
    position: relative; // 因为dragDiv设置了绝对定位，是基于最近的设置了绝对或者相对定位的元素进行定位
    &_top {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0 1.5vh;
        margin-bottom: 1.5vh;
        &_title {
            font-size: 1.6vh;
            font-weight: 600;
            color: #000000;
        }
        &_rightBtn {
            cursor: pointer;
            font-size: 2vh;
            display: flex;
        }
    }
}

.dragDiv {
    width: 20vh;
    height: fit-content;
    position: absolute;
    right: 2%;
    z-index: 3;
    background: #fff;
    display: flex;
    flex-direction: column;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
    border-radius: 0.5vh;
    padding: 1vh;
    &_top {
        display: flex;
        justify-content: space-between;
        &_reset {
            color: #1677ff;
            cursor: pointer;
        }
        &_reset:hover{
            color: #7fbdff;
        }
    }
    .ivu-divider-horizontal {
        margin: 0.8vh 0 0.5vh 0;
    }
    &_item {
        cursor: pointer;
    }
}

.pageDiv {
    width: 100%;
    margin-top: 0.8vh;
    display: flex;
    flex-direction: row-reverse;
}
</style>
