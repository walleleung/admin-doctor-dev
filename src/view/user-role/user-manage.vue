<style lang="less" scoped>
	@lightColor: #999999;
	.detail {
		&_base{
			position: relative;
			&_img{
				height: 80px;
				width: 80px;
			}
			&_name{
				position: absolute;
				left: 100px;
				top: 20px;
				width: calc(~'100% - 100px');
				h3{
					display: inline-block;
				}
				span{
					margin-left: 8px;
					color: @lightColor;
				}
			}
			&_item{
				padding-bottom: 15px;
				label{
					color: @lightColor;
					width: 100px;
					text-align: right;
					display: inline-block;
				}
				img{
					height: 60px;
					width: 60px;
				}
			}
		}
		&_wrapper{
			border-top: 1px solid #eeeeee;
			&_title{
				font-weight: bold;
				font-size: 14px;
				padding: 10px 0;
				color: #515a6e;
			}
		}
		&_role{
			color: #348EED;
			padding-bottom: 20px;
		}
	}
</style>

<template>
	<div class="user">
		<div class="user_main">
			<Row class="user_main_search common_search" :gutter="20">
				<Col span="6">
					<span>用户名</span>
					<Input v-model="searchParams.userName"></Input>
				</Col>
				<Col span="6">
					<span>真实姓名</span>
					<Input v-model="searchParams.realName"></Input>
				</Col>
				<Col span="6">
					<span>状态</span>
					<Select v-model="searchParams.isEnable">
						<Option value=''>全部</Option>
						<Option value='0'>启用</Option>
						<Option value='1'>禁用</Option>
					</Select>
				</Col>
				<Col span="2">
					<Button @click="handlePage(1)" type="primary" :loading="tableParams.isLoading">查询</Button>
					<Button @click="handleAddEdit('')" v-permission="this.$API.userManage.create">添加用户</Button>
				</Col>
			</Row>
			<Card>
				<div class="user_main_list">
					<Table :columns="config" :data="tableParams.data" :loading="tableParams.isLoading"></Table>
				</div>
				<Row class="user_main_page">
					<Page :total="tableParams.total" :page-size="searchParams.limit" show-total :current="searchParams.pager" @on-change="handlePage"></Page>
				</Row>
			</Card>
		</div>
		<Drawer
			title="查看用户"
			class="detail"
			v-model="isDrawerShow"
			width="30"
			:mask-closable="false">
			<div>
				<!--基本信息-->
				<div class="detail_base">
					<img class="detail_base_img" :src="detailData.upmsUser.logo">
					<div class="detail_base_name">
						<div>
							<h3>{{detailData.upmsUser.userName}}</h3>
							<span>(用户名)</span>
						</div>
						<div>
							<h3>{{detailData.upmsUser.realName}}</h3>
							<span>(真实姓名)</span>
						</div>
					</div>
					<div class="detail_base_item">
						<label>联系电话：</label>{{detailData.upmsUser.phone}}
					</div>
					<div class="detail_base_item">
						<label>专属号码：</label>{{detailData.upmsUser.mobile}}
					</div>
					<div class="detail_base_item">
						<label>职位：</label>{{detailData.upmsUser.position}}
					</div>
					<div class="detail_base_item">
						<label>职称：</label>{{detailData.upmsUser.title}}
					</div>
					<div class="detail_base_item">
						<label>简介：</label>{{detailData.upmsUser.introduce}}
					</div>
					<div class="detail_base_item">
						<label>主要科室：</label>{{detailData.upmsUser.orgainzationName}}
					</div>
					<div class="detail_base_item">
						<label>医疗机构：</label>{{detailData.upmsUser.medicalName}}
					</div>
					<div class="detail_base_item">
						<label>是否禁用：</label>{{detailData.upmsUser.isEnable == 0 ? '启用': '禁用'}}
					</div>
					<div class="detail_base_item" v-for="item in detailData.upmsFieldDynamicList" :key="item.fieldId">
						<label>{{item.fieldName}}：</label>
						<template v-if="item.fieldType!=='image'">{{item.fieldValue}}</template>
						<img v-else :src="item.fieldValue">
					</div>
				</div>
				<div class="detail_wrapper detail_role">
					<div class="detail_wrapper_title">角色</div>
					<template v-for="(item, index) in detailData.roleList">{{item.roleName}}<Divider type="vertical" v-show="index!==detailData.roleList.length-1" /></template>
				</div>
				<div class="detail_wrapper">
					<div class="detail_wrapper_title">组织数据权限</div>
					<Tree :data="detailData.organizationList"></Tree>
				</div>
				<div class="detail_wrapper">
					<div class="detail_wrapper_title">关联用户数据</div>
					<Table :columns="drawerUserTableConfig" :data="detailData.userList"></Table>
				</div>
				<div class="detail_wrapper">
					<div class="detail_wrapper_title">方案</div>
					<Table :columns="drawerSchemeTableConfig" :data="detailData.userSchemeList"></Table>
				</div>
				<div class="detail_wrapper">
					<div class="detail_wrapper_title">疾病</div>
					<Table :columns="drawerDiseaseTableConfig" :data="detailData.userDiseaseList"></Table>
				</div>
				<div style="padding: 20px 80px 0 80px;">
					<Button type="primary" @click="handleAddEdit(detailData.upmsUser.userId, 'edit')" v-permission="this.$API.userManage.update">编辑</Button>
					<Button type="error" @click="handleDelete(detailData.upmsUser.userId)" style="float: right;" v-permission="this.$API.userManage.delete">删除</Button>
				</div>
			</div>
		</Drawer>
	</div>
</template>

<script>
import { formatToTreeData, handleDeleteRefresh } from '@/libs/businessUtil'
export default {
    name: 'user-manage',
    data () {
        return {
            isDrawerShow: false,
            tableParams: {
                data: [],
                total: 0,
                isLoading: false
            },
            searchParams: {
                userName: '',
                realName: '',
                isEnable: '',
                limit: 10,
                sort: '', // 排序字段
                order: '' // 排序方式:降序 DESC 升序 ASC
            },
            config: [
                {
                    title: '用户名',
                    key: 'userName'
                },
                {
                    title: '真实姓名',
                    key: 'realName'
                },
                {
                    title: '联系电话',
                    key: 'mobile'
                },
                {
                    title: '状态',
                    key: 'isEnable',
                    render: (h, params) => {
                        return h('span', params.row.isEnable !== 0 ? '禁用' : '启用')
                    }
                },
                {
                    title: '操作',
                    key: 'action',
                    width: 180,
                    render: (h, params) => {
                        return h('div', [
                            h(
                                'Button',
                                {
                                    props: {
                                        type: 'text',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    directives: [
                                        {
                                            name: 'permission',
                                            value: this.$API.userManage.allInfo
                                        }
                                    ],
                                    on: {
                                        click: () => {
                                            this.handleDetail(params.row.userId)
                                        }
                                    }
                                },
                                '查看'
                            ),
                            h(
                                'Button',
                                {
                                    props: {
                                        type: 'text',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    directives: [
                                        {
                                            name: 'permission',
                                            value: this.$API.userManage.update
                                        }
                                    ],
                                    on: {
                                        click: () => {
                                            this.handleAddEdit(params.row.userId)
                                        }
                                    }
                                },
                                '编辑'
                            ),
                            h(
                                'Button',
                                {
                                    props: {
                                        type: 'text',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    directives: [
                                        {
                                            name: 'permission',
                                            value: this.$API.userManage.delete
                                        }
                                    ],
                                    on: {
                                        click: () => {
                                            this.handleDelete(params.row.userId)
                                        }
                                    }
                                },
                                '删除'
                            )
                        ])
                    }
                }
            ],
            detailData: {
                organizationList: [],
                roleList: [],
                upmsFieldDynamicList: [],
                upmsUser: {},
                userList: []
            }, // 查看块所有数据
            drawerUserTableConfig: [
                {
                    title: '用户真实姓名',
                    key: 'realName',
                    align: 'center'
                },
                {
                    title: '所属组织',
                    key: 'orgainzationName',
                    align: 'center'
                }
            ],
            drawerSchemeTableConfig: [
                {
                    title: '方案名称',
                    key: 'schemeName',
                    align: 'center'
                },
                {
                    title: '所属组织',
                    key: 'organizationName',
                    align: 'center'
                }
            ],
            drawerDiseaseTableConfig: [
                {
                    title: '名称',
                    key: 'diseaseName',
                    align: 'center'
                },
                {
                    title: '类型',
                    key: 'diseaseTypeName',
                    align: 'center'
                }
            ]
        }
    },
    methods: {
        async handleQuery () {
        	this.tableParams.isLoading = true
            const res = await this.$API.userManage.list(this.searchParams).catch(e => {
                this.tableParams.isLoading = false
            })
            this.tableParams.data = res.data
            this.tableParams.total = res.total
            this.tableParams.isLoading = false
        },
        handlePage (pager) {
            this.searchParams.pager = pager
            this.handleQuery()
        },
        /**
		 * @description 添加、编辑按钮调用方法
		 * @param rowData
		 * @param type
		 */
        handleAddEdit (userId) {
            this.$router.push({
                path: '/user-role/user-edit',
                query: { userId }
            })
            this.isDrawerShow = false
        },
        /**
		 * @description 查看按钮调用方法
		 * @param rowData
		 */
        async handleDetail (id) {
            const res = await this.$API.userManage.allInfo({ id: id })
            res.data.upmsUser.organizationList = formatToTreeData({
                baseArr: res.data.upmsUser.organizationList,
                parentIdName: 'parentId',
                idName: 'organizationId',
                sortName: 'sort',
                childrenFn: function (item) {
                    return {
                        value: item.organizationId,
                        organizationId: item.organizationId,
                        title: item.name,
                        expand: true,
                        children: []
                    }
                }
            })
            this.detailData = res.data.upmsUser
            this.isDrawerShow = true
        },
        /**
		 * @description 删除 操作
		 * @param delId
		 */
        handleDelete (delId) {
            this.$Modal.confirm({
                title: '删除提示',
                content: '确定要删除该用户吗?',
                onOk: async () => {
                    await this.$API.userManage.delete({ ids: delId })
                    this.isDrawerShow = false
                    handleDeleteRefresh('searchParams', 'tableParams', 'handleQuery', this)
                }
            })
        }
    },
    mounted () {
        this.handleQuery()
    }
}
</script>
