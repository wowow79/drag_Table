<template>
    <div>
        <tableCard
            v-model="tableData"
            title="数据列表"
            :columns="tableColumns"
            :current="tableCurrent"
            :total="tableTotal"
            :pageSize="tablePageSize"
            :height="500"
            @refreash="tableRefreash"
            @changePage="tableChangePage"
            @changeSize="tableChangeSize"
            @editBtn="editBtn"
            @delBtn="delBtn">
            <Button icon="md-add" slot="btnSlot" style="marginRight: 1vh;">新建</Button>
        </tableCard>
    </div>
</template>

<script>
    import tableCard from '@/components/tableCard'
    export default {
        name: '',
        props: {
        },
        components: {
            tableCard
        },
        data () {
            return {
                tableData: [],
                tableColumns: [
                    {
                        title: '序号',
                        type: 'index',
                        align: 'center',
                        width: 70
                    },
                    {
                        title: '描述',
                        key: 'describe',
                        align: 'center'
                    },
                    {
                        title: '创建时间',
                        key: 'createdTime',
                        align: 'center'
                    },
                    {
                        title: '操作',
                        key: 'action',
                        align: 'center',
                        width: 190,
                        needRender: true, // 需要render函数的列必须有这个
                        button: [ // 自定义列，render的内容，用button替换原本的render，内容书写方式不变，<template>那种写法在这个组件失效
                            (h, p, vm) => {
                                return h('div', [
                                    h('Button', {
                                        props: {
                                            size: 'small'
                                        },
                                        style: {
                                            marginRight: '0.5vh',
                                            border: '0.1vh solid #2d8cf0',
                                            color: '#2d8cf0'
                                        },
                                        on: {
                                            click: () => {
                                                vm.$emit('editBtn', p.row)
                                            }
                                        }
                                    }, '编辑'),
                                    h('Poptip', {
                                        props: {
                                            transfer: true,
                                            confirm: true,
                                            title: '确认删除？'
                                        },
                                        style: {
                                            marginRight: '0.5vh'
                                        },
                                        on: {
                                            'on-ok': () => {
                                                vm.$emit('delBtn', p.row)
                                            },
                                            'on-cancel': () => {
                                                vm.$Message.info('点击了取消')
                                            }
                                        }
                                    }, [
                                        h('Button', {
                                            props: {
                                                size: 'small'
                                            },
                                            style: {
                                                border: '0.1vh solid red',
                                                color: 'red'
                                            }
                                        }, '删除')
                                    ])
                                ])
                            }
                        ]
                    }
                ],
                tableCurrent: 1,
                tableTotal: 0,
                tablePageSize: 10
            }
        },
        created () {
            this.getList()
        },
        mounted () {

        },
        methods: {
            getList () {
                this.tableData = [
                    {
                        describe: '这个人很懒，什么也没留下。',
                        createdTime: '2024-04-19 15:11:11'
                    }
                ]
            },
            tableRefreash () {
                this.tableCurrent = 1
                this.getList()
            },
            tableChangePage (e) {
                this.tableCurrent = e
                this.getList()
            },
            tableChangeSize (e) {
                this.tablePageSize = e
                this.getList()
            },
            editBtn (row) {

            },
            delBtn (row) {

            }
        }
    }
</script>

<style scoped>

</style>
