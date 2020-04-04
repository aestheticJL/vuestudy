<template>
    <div>
        <el-input
                size="small"
                style="width: 300px;margin-right: 13px"
                placeholder="请添加职称"
                prefix-icon="el-icon-plus"
                v-model="Job.name">
        </el-input>
        <el-select v-model="Job.titleLevel" placeholder="请选择职称" size="small" style="margin-right: 13px">
            <el-option
                    v-for="item in titleLevels"
                    :key="item"
                    :label="item"
                    :value="item">
            </el-option>
        </el-select>
        <el-button type="primary" size="small" prefix-icon="el-icon-plus" @click="addJobLevel">添加</el-button>
        <el-table
                v-loading="loading"
                element-loading-text="加载中"
                element-loading-spinner="el-icon-loading"
                element-loading-background="rgba(0, 0, 0, 0.8)"
                border
                stripe
                ref="multipleTable"
                :data="jobLevel"
                tooltip-effect="dark"
                @keydown.enter.native="addJobLevel"
                @selection-change="handleSelectionChange"
                style="width: 80%;margin-top:10px">
            <el-table-column
                    type="selection"
                    width="55">
            </el-table-column>
            <el-table-column
                    prop="id"
                    label="编号"
                    width="55">
            </el-table-column>
            <el-table-column
                    prop="name"
                    label="职称名称"
                    width="200">
            </el-table-column>
            <el-table-column
                    prop="titleLevel"
                    label="职称级别"
                    width="200">
            </el-table-column>
            <el-table-column
                    prop="createDate"
                    label="创建时间">
            </el-table-column>
            <el-table-column
                    label="是否启用">
                <template slot-scope="scope">
                    <el-tag size="small" type="success" v-if="scope.row.enabled">已启用</el-tag>
                    <el-tag size="small" type="danger" v-else>未启用</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            @click="handleEdit(scope.row)">编辑
                    </el-button>
                    <el-button
                            size="mini"
                            type="danger"
                            @click="handleDelete(scope.row)">删除
                    </el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-button
                size="mini"
                type="danger"
                style="margin-top: 13px"
                @click="deleteSomeJobLevel(multipleSelection)"
                :disabled="multipleSelection.length===0"
        >批量删除
        </el-button>
        <el-dialog
                title="修改职称"
                :visible.sync="dialogVisible"
                width="60%">
            <div>
                <div>
                    <el-tag>职位名称</el-tag>
                    <el-input size="small" class="updateJobInput" v-model="updateJob.name" style="margin-right: 8px">
                        {{updateJob.name}}
                    </el-input>
                    <el-select v-model="updateJob.titleLevel" placeholder="请选择职称" size="small">
                        <el-option
                                v-for="item in titleLevels"
                                :key="item"
                                :label="item"
                                :value="item">
                        </el-option>
                    </el-select>
                </div>
                <div style="margin-top: 13px">
                    <el-switch
                            style="display: block"
                            v-model="updateJob.enabled"
                            active-color="#13ce66"
                            inactive-color="#ff4949"
                            active-text="启用"
                            inactive-text="禁用">
                    </el-switch>
                </div>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button size="small" @click="dialogVisible = false">取 消</el-button>
    <el-button size="small" type="primary" @click="updateJobLevel">确 定</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "JobLevelMana",
        data() {
            return {
                Job: {
                    name: '',
                    titleLevel: ''
                },
                updateJob: {
                    name: '',
                    titleLevel: '',
                    enabled: false
                },
                titleLevels: [
                    "正高级",
                    "副高级",
                    "中级",
                    "初级",
                    "员级",
                ],
                jobLevel: [],
                multipleSelection: [],
                dialogVisible: false,
                loading: false,
            }
        },
        mounted() {
            this.initJobLevel();
        },
        methods: {
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            initJobLevel() {
                this.loading = true;
                this.getRequest("/system/basic/joblevel/").then(resp => {
                    this.loading = false;
                    if (resp) {
                        this.jobLevel = resp;
                    }
                })
            },
            addJobLevel() {
                if (this.Job.name) {
                    this.postRequest("/system/basic/joblevel/", this.Job).then(resp => {
                        if (resp) {
                            this.initJobLevel();
                            this.Job.name = '';
                            this.Job.titleLevel = '';
                        }
                    })
                } else {
                    this.$message.error('职称名不可以为空');
                }
            },
            handleEdit(data) {
                Object.assign(this.updateJob, data);
                this.dialogVisible = true;
            },
            handleDelete(data) {
                this.$confirm('此操作将永久删除' + data.name + '职称, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/basic/joblevel/" + data.id).then(resp => {
                        if (resp) {
                            this.initJobLevel();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            updateJobLevel() {
                this.putRequest("/system/basic/joblevel/", this.updateJob).then(resp => {
                    if (resp) {
                        this.initJobLevel();
                        this.updateJob.name = '';
                        this.updateJob.titleLevel = '';
                    } else {
                        this.$message.error('修改出错');
                    }
                });
                this.dialogVisible = false;
            },
            deleteSomeJobLevel() {
                this.$confirm('此操作将永久删除所有选中的职称, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let ids = "?";
                    this.multipleSelection.forEach(item => {
                        ids += 'ids=' + item.id + '&'
                    });
                    this.deleteRequest("/system/basic/joblevel/" + ids).then(resp => {
                        if (resp) {
                            this.initJobLevel();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            }
        }
    }
</script>

<style scoped>
    .updateJobInput {
        width: 300px;
        margin-left: 8px;
    }
</style>