<template>
    <div>
        <el-input
                size="small"
                style="width: 300px;margin-right: 13px"
                placeholder="请添加职位"
                prefix-icon="el-icon-plus"
                v-model="pos.name">
        </el-input>
        <el-button type="primary" size="small" prefix-icon="el-icon-plus" @click="addPosition">添加职位</el-button>
        <el-table
                :data="Position"
                border
                stripe
                @keydown.enter.native="addPosition"
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
                    label="职位名称"
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
                            @click="handleEdit(scope.$index, scope.row)">编辑
                    </el-button>
                    <el-button
                            size="mini"
                            type="danger"
                            @click="handleDelete(scope.$index, scope.row)">删除
                    </el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-button
                size="mini"
                type="danger"
                style="margin-top: 13px"
                @click="deleteSomePosition(multipleSelection)"
                :disabled="multipleSelection.length===0"
        >批量删除
        </el-button>
        <el-dialog
                title="修改职位"
                :visible.sync="dialogVisible"
                width="30%">
            <div>
                <div>
                    <el-tag>职位名称</el-tag>
                    <el-input size="small" class="updatePosInput" v-model="updatePos.name">{{updatePos.name}}</el-input>
                </div>
                <div style="margin-top: 13px">
                        <el-switch
                                style="display: block"
                                v-model="updatePos.enabled"
                                active-color="#13ce66"
                                inactive-color="#ff4949"
                                active-text="启用"
                                inactive-text="禁用">
                        </el-switch>
                </div>
            </div>
            <span slot="footer" class="dialog-footer">
    <el-button size="small" @click="dialogVisible = false">取 消</el-button>
    <el-button size="small" type="primary" @click="updatePosition">确 定</el-button>
  </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "PosMana",
        data() {
            return {
                pos: {
                    name: ''
                },
                updatePos: {
                    name: '',
                    enabled:false
                },
                Position: [],
                multipleSelection: [],
                dialogVisible: false
            }
        },
        mounted() {
            this.initPosition();
        },
        methods: {
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            initPosition() {
                this.getRequest("/system/basic/pos/").then(resp => {
                    if (resp) {
                        this.Position = resp;
                    }
                })
            },
            addPosition() {
                if (this.pos.name) {
                    this.postRequest("/system/basic/pos/", this.pos).then(resp => {
                        if (resp) {
                            this.initPosition();
                            this.pos.name = "";
                        }
                    })
                } else {
                    this.$message.error('职位名不可以为空');
                }
            },
            updatePosition() {
                this.putRequest("/system/basic/pos/", this.updatePos).then(resp => {
                    if (resp) {
                        this.initPosition();
                        this.updatePos.name = '';
                    } else {
                        this.$message.error('修改出错');
                    }
                });
                this.dialogVisible = false;
            },
            handleEdit(index, data) {
                Object.assign(this.updatePos, data);
                this.dialogVisible = true;
            },
            handleDelete(index, data) {
                this.$confirm('此操作将永久删除' + data.name + '职位, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/basic/pos/" + data.id).then(resp => {
                        if (resp) {
                            this.initPosition();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            deleteSomePosition() {
                this.$confirm('此操作将永久删除所有选中的职位, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let ids = "?";
                    this.multipleSelection.forEach(item => {
                        ids += 'ids=' + item.id + '&'
                    });
                    this.deleteRequest("/system/basic/pos/" + ids).then(resp => {
                        if (resp) {
                            this.initPosition();
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
    .updatePosInput {
        width: 300px;
        margin-left: 8px;
    }
</style>