<template>
    <div>
        <div>
            <el-input v-model="empSearch" placeholder="请输入员工姓名" style="width: 400px;margin-right: 13px"
                      @keydown.enter.native="initEmployee"/>
            <el-button type="primary" @click="initEmployee">搜索</el-button>
            <el-button type="primary" disabled>高级搜索</el-button>
            <el-button type="primary" class="rightButton" icon="el-icon-plus" disabled>添加员工</el-button>
            <el-button type="success" class="rightButton" icon="el-icon-top" disabled>导出数据</el-button>
            <el-button type="success" class="rightButton" icon="el-icon-bottom" disabled>导入数据</el-button>
        </div>
        <div>
            <template>
                <el-table
                        v-loading="loading"
                        element-loading-text="加载中"
                        element-loading-spinner="el-icon-loading"
                        element-loading-background="rgba(0, 0, 0, 0.8)"
                        :data="employee"
                        border
                        stripe
                        style="width: 100%;margin-top: 20px;">
                    <el-table-column
                            prop="name"
                            label="姓名"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="gender"
                            label="性别"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="birthday"
                            label="生日"
                            width="250">
                    </el-table-column>
                    <el-table-column
                            prop="idCard"
                            label="身份证号码"
                            width="250">
                    </el-table-column>
                    <el-table-column
                            prop="wedlock"
                            label="是否已婚"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="nName"
                            label="民族"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="nativePlace"
                            label="籍贯"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="pName"
                            label="政治面貌"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="email"
                            label="邮箱"
                            width="200">
                    </el-table-column>
                    <el-table-column
                            prop="phone"
                            label="手机号码"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="address"
                            label="住址"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="dName"
                            label="部门"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="jName"
                            label="职称"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="posName"
                            label="职位"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="engageForm"
                            label="聘用形式"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="tiptopDegree"
                            label="学历"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="specialty"
                            label="专业"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="school"
                            label="大学"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="beginDate"
                            label="入职日期"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="workState"
                            label="是否在职"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="workID"
                            label="工号"
                            width="120">
                    </el-table-column>
                    <el-table-column
                            prop="contractTerm"
                            label="合同期限"
                            width="75">
                    </el-table-column>
                    <el-table-column
                            prop="conversionTime"
                            label="转正日期"
                            width="250">
                    </el-table-column>
                    <el-table-column
                            prop="beginContract"
                            label="合同起始日期"
                            width="250">
                    </el-table-column>
                    <el-table-column
                            prop="endContract"
                            label="合同截至日期"
                            width="250">
                    </el-table-column>
                    <el-table-column
                            fixed="right"
                            label="操作"
                            width="300">
                        <template slot-scope="scope">
                            <el-button
                                    disabled
                                    type="primary"
                                    size="mini">
                                编辑
                            </el-button>
                            <el-button
                                    disabled
                                    type="success"
                                    size="mini">
                                查看高级资料
                            </el-button>
                            <el-button
                                    @click="deleteEmp(scope.row)"
                                    type="danger"
                                    size="mini">
                                删除
                            </el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </template>
            <el-pagination
                    class="page"
                    background
                    layout="prev, pager, next, jumper, ->, total"
                    :total="total"
                    @current-change="currentChange">
            </el-pagination>
        </div>
    </div>
</template>

<script>
    export default {
        name: "EmpBasic",
        data() {
            return {
                employee: [
                    {
                        "name": "wo"
                    }
                ],
                total: 0,
                page: 1,
                size: 10,
                empSearch: '',
                loading: false
            };
        },
        mounted() {
            this.initEmployee();
        },
        methods: {
            initEmployee() {
                this.loading = true;
                this.getRequest("/employee/basic/?page=" + this.page + "&size=" + this.size + "&key=" + this.empSearch).then(resp => {
                    this.loading = false;
                    if (resp) {
                        this.employee = resp.data;
                        this.total = resp.total;
                    }
                })
            },
            currentChange(currentPage) {
                this.page = currentPage;
                this.initEmployee();
            },
            deleteEmp(data) {
                this.$confirm('此操作将永久删除' + data.name + '员工, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/employee/basic/" + data.id).then(resp => {
                        if (resp) {
                            this.initEmployee();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
        }
    }
</script>

<style>
    .rightButton {
        float: right;
    }

    .page {
        display: flex;
        justify-content: center;
        margin-top: 13px;
    }
</style>