<template>
    <div class="animated fadeIn">

        <Row>
            <Col :md="24">

            <div>

                <div id="container" style="margin-bottom:10px;">

                    <Button type="primary" size="large" icon="ios-cloud-upload-outline" id='pickfiles' @click="qiniu_upload" style="padding-bottom:5px;">上传文件</Button>
                    <Progress :percent="progresscount" :status="progresstatus" style="width=90%;vertical-align:middle" v-if="progresshow">
                        <Icon type="checkmark-circled" v-if="progresscount==100"></Icon>
                        <span v-if="progresscount===100">上传成功</span>
                        <Icon type="close-circled" v-if="progresstatus==='wrong'"></Icon>
                        <span v-if="progresstatus==='wrong'">上传失败</span>
                        <span style="position:absolute;left:50%;top:-5px;" v-if="progresscount!==0&&progresscount!=100">{{progressspeed}}</span>
                    </Progress>
                </div>

                <div style="position:relative;">

                    <Table :columns="columns7" :data="page_video_list" ref="table"></Table>

                    <div style="position:absolute;top:0px;width:100%;height:100%;display: flex;
                                        align-items: center;
                                        justify-content: center;background: rgba(210, 216, 222, 0.5);" v-if="list_loadding">
                        <Spin size="large"></Spin>
                        <h6 style="color:#2d8cf0;margin-top:10px;">正在获取数据...</h6>
                    </div>
                </div>

                <Page :total="this.video_list.length" show-total @on-change="setInitPage" style="text-align:right;margin-top:50px"></Page>

                <Button type="primary" size="large" @click="exportData(1)">
                    <Icon type="ios-download-outline"></Icon> 导出原始数据</Button>
                <Button type="primary" size="large" @click="exportData(2)">
                    <Icon type="ios-download-outline"></Icon> 导出排序和过滤后的数据</Button>
            </div>
            

            </Col>

            <Col :md="12">

            </Col>

        </Row>

    </div>
</template>

<script>
import expandRow from './table-expand.vue';

var vue;
var uploader;
export default {
    components: { expandRow },

    name: 'buttons',
    data() {
        return {
            tableData: [],
            apiUrl: 'http://127.0.0.1:8444/api/persons',
            total: 0,
            pageSize: 10,
            currentPage: 1,
            sex: '',
            email: '',
            dialogFormVisible: false,
            form: '',
            progresshow: false,
            progresscount: 0,
            progresstatus: 'active',
            progressspeed: 0,
            video_list: [
                {
                    task_status: 0,
                    timestamp: "2017-08-10 17:03:54",
                    video_duration: "11s",
                    video_size: 1732410,
                    video_name: "p.mp4",
                },
                {
                    task_status: 1,
                    timestamp: "2017-08-10 17:03:54",
                    video_duration: "11s",
                    video_size: 1732410,
                    video_name: "p.mp4",
                },
                {
                    task_status: 1,
                    timestamp: "2017-08-10 17:03:54",
                    video_duration: "11s",
                    video_size: 1732410,
                    video_name: "p.mp4",
                },
                {
                    task_status: 0,
                    timestamp: "2017-08-10 17:03:54",
                    video_duration: "11s",
                    video_size: 1732410,
                    video_name: "p.mp4",
                },
                {
                    task_status: 1,
                    timestamp: "2017-08-10 17:03:54",
                    video_duration: "11s",
                    video_size: 1732410,
                    video_name: "p.mp4",
                },
                {
                    task_status: 0,
                    timestamp: "2017-08-10 17:03:54",
                    video_duration: "11s",
                    video_size: 1732410,
                    video_name: "p.mp4",
                },
            ],
            pageindex: 1,
            page_video_list: [],
            lodding: false,
            list_loadding: false,
            columns7: [
                {
                    type: 'expand',
                    width: "30",
                    ellipsis: 'true',

                    render: (h, params) => {
                        return h(expandRow, {
                            props: {
                                row: params.row
                            }
                        })
                    }
                },
                {
                    title: '项目ID',
                    key: 'video_name',
                    ellipsis: 'true',
                },
                {
                    title: '项目名称',
                    ellipsis: 'true',
                    key: 'timestamp',

                },

                {
                    title: '金额',
                    ellipsis: 'true',
                    key: 'video_duration',
                },
                {
                    title: '负责人',
                    ellipsis: 'true',
                    key: 'video_size',
                    filters: [
                        {
                            label: '大于10M',
                            value: 1
                        },
                        {
                            label: '小于10M',
                            value: 2
                        }
                    ],
                    filterMultiple: false,
                    filterMethod(value, row) {
                        if (value === 1) {
                            return row.video_size > 1024 * 1024 * 1024 * 10;
                        } else if (value === 2) {
                            return row.video_size < 1024 * 1024 * 10;
                        }
                    }
                },
                {
                    title: '状态',
                    ellipsis: 'true',
                    filters: [
                        {
                            label: '处理完成',
                            value: 1
                        },
                        {
                            label: '正在处理',
                            value: 2
                        }
                    ],
                    filterMultiple: false,
                    filterMethod(value, row) {
                        if (value === 1) {
                            return row.task_status === 1;
                        } else if (value === 2) {
                            return row.task_status === 0;
                        }
                    },

                    render: (h, params) => {
                        const task_status = parseInt(params.row.task_status);

                        if (task_status === 0)
                            return h('div', [
                                h('Tag', {
                                    props: {
                                        type: 'dot',
                                        color: "blue"
                                    }
                                }, "正在处理"),
                            ]);

                        else if (task_status === 1)
                            return h('div', [

                                h('Tag', {
                                    props: {
                                        type: 'dot',
                                        color: "green"
                                    }
                                }, "处理完成"),
                            ]);
                    }
                },

                {
                    title: '操作',
                    key: 'action',
                    align: 'center',
                    ellipsis: 'true',

                    render: (h, params) => {
                        const task_status = parseInt(params.row.task_status);
                        if (task_status === 0) {
                            return h('div', [
                                h('Tooltip', {
                                    props: {
                                        content: '还未分析完成，暂时不能查看',
                                    },
                                }, [
                                        h('Button', {
                                            props: {
                                                type: 'primary',
                                                size: 'small',
                                                loading: true,
                                            },
                                            style: {
                                                marginRight: '5px'
                                            },
                                        }, '处理中')]),
                                h('Button', {
                                    props: {
                                        type: 'error',
                                        size: 'small',
                                        disabled: true,
                                    },
                                    on: {
                                        click: () => {
                                            this.remove(params.index)
                                        }
                                    }
                                }, '删除')


                            ]);
                        }
                        else if (task_status === 1) {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small',
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    on: {
                                        click: () => {
                                            this.$router.push({ path: '/tabledetail/' + (((this.pageindex - 1) * 10) + params.index) })

                                        }
                                    },
                                }, '查看结果'),
                                h('Button', {
                                    props: {
                                        type: 'error',
                                        size: 'small',
                                    },
                                    on: {
                                        click: () => {
                                            this.remove(params.index)
                                        }
                                    }
                                }, '删除')
                            ]);
                        }
                    }//render
                },//{
            ],//cloumn
        }//return
    },//data
    methods: {
        remove(index) {
            this.video_list.splice(index, 1);
        },
        qiniu_upload() {
            uploader.start();
        },
        setInitPage(page) {

            this.page_video_list = [];
            this.pageindex = page;
            let video_list = this.video_list;
            for (let i = (page - 1) * 10; i < (page - 1) * 10 + 10; i++) {


                if (i < video_list.length) {

                    this.page_video_list.push(this.video_list[i]);

                }
            }
            console.log("page_video_list："+ this.page_video_list)
        },
        exportData(type) {
            if (type === 1) {
                this.$refs.table.exportCsv({
                    filename: '原始数据',
                    columns: this.columns7,
                    data: this.video_list
                });
            } else if (type === 2) {
                this.$refs.table.exportCsv({
                    filename: '排序和过滤后的数据',
                    original: false
                });
            }
        },
        getCustomers() {
            this.$axios.get('http://127.0.0.1:8444/api/persons', {
                params: {
                    page: 1,
                }
            }).then((response) => {
                // this.tableData = response.data.data.results;
                // this.total = response.data.data.total;
                // this.pageSize = response.data.data.count;
                console.log("表格数据" + response.data.data);

            }).catch(function(response) {
                console.log('````````````````````````custom is :' + response)
            });
        }
    },
    mounted() {
        //表格的mounted
        const vue = this;
      

        this.list_loadding = true;
        setTimeout(function() {
            vue.list_loadding = false;

        }, 2000)
        // qiniuInit(vue);//初始化七牛数据
        this.setInitPage(1);
        // this.getCustomers()
    }
}
</script>


<style type="text/css" scoped>
.ivu-tag-dot {
    border: none!important;
}

tr.ivu-table-row-hover td .ivu-tag-dot {
    background-color: #ebf7ff!important;
}

.demo-i-circle-custom h1 {
    color: #3f414d;
    font-size: 10px;
    font-weight: normal;
}

.demo-i-circle-custom p {
    color: #657180;
    font-size: 8px;
    margin: 5px 0 2px;
}

.demo-i-circle-custom span {
    display: block;
    padding-top: 15px;
    color: #657180;
    font-size: 10px;
}

.demo-i-circle-custom span :before {
    content: '';
    display: block;
    width: 50px;
    height: 1px;
    margin: 0 auto;
    background: #e0e3e6;
    position: relative;
    top: -20px;
}

;
.demo-i-circle-custom span i {
    font-style: normal;
    color: #3f414d;
}


/*wz-btn wz-btn-primary wz-btn-small wz-btn-loading*/

.ivu-btn.ivu-btn-primary.ivu-btn-small:not(.ivu-btn-loading) {
    padding: 2px 10px!important;
}

td.ivu-table-expanded-cell {
    background-color: white!important;
}
</style>