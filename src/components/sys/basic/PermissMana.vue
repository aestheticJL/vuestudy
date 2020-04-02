<template>
    <div>
        <div>
            <el-input placeholder="请输入英文名称" v-model="role.name" style="width: 25%" size="small">
                <template slot="prepend">ROLE_</template>
            </el-input>
            <el-input v-model="role.nameZh"
                      placeholder="请输入中文名称"
                      style="width: 25%;margin-left: 13px;margin-right: 13px"
                      size="small">
            </el-input>
            <el-button type="primary" size="small" prefix-icon="el-icon-plus" @click="addRole">添加角色</el-button>
        </div>
        <el-collapse accordion style="margin-top: 13px;width: 46%" @change="change" v-model="activeName">
            <el-collapse-item :title="item.nameZh" :name="item.id" v-for="(item,index) in roles" :key="index">
                <div>
                    <el-card class="box-card">
                        <div slot="header">
                            <span>角色权限</span>
                            <el-button style="float: right; padding: 3px 0" type="text" icon="el-icon-delete" @click="deleteRole(item)"/>
                        </div>
                        <div>
                            <el-tree :data="allMenus"
                                     :props="defaultProps"
                                     show-checkbox node-key="id"
                                     :key="index"
                                     :default-checked-keys="selectedMenus"
                                     ref="tree"/>
                            <div style="display: flex;justify-content: flex-end">
                                <el-button @click="cancelUpdate">取消修改</el-button>
                                <el-button type="primary" @click="doUpdate(item.id,index)">确认修改</el-button>
                            </div>
                        </div>
                    </el-card>
                </div>
            </el-collapse-item>
        </el-collapse>
    </div>
</template>

<script>

    export default {
        name: "permissMana",
        data() {
            return {
                role: {
                    name: '',
                    nameZh: ''
                },
                allMenus: [],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                selectedMenus: [],
                roles: [],
                activeName: -1
            }
        },
        mounted() {
            this.initRoles();
        },
        methods: {
            initRoles() {
                this.getRequest("/system/basic/permiss/").then(resp => {
                    if (resp) {
                        this.roles = resp;
                    }
                })
            },

            change(rid) {
                if (rid) {
                    this.initAllMenus();
                    this.initSelectedMenus(rid);
                }
            },
            initSelectedMenus(rid) {
                this.getRequest("/system/basic/permiss/mids/" + rid).then(resp => {
                    if (resp) {
                        this.selectedMenus = resp;
                    }
                })
            },
            initAllMenus() {
                this.getRequest("/system/basic/permiss/menus").then(resp => {
                    if (resp) {
                        this.allMenus = resp;
                    }
                })
            },
            doUpdate(rid, index) {
                let tree = this.$refs.tree[index];
                let checkedKeys = tree.getCheckedKeys(true);
                let url = "/system/basic/permiss/?rid=" + rid;
                checkedKeys.forEach(t => {
                    url += '&mids=' + t;
                });
                this.putRequest(url).then(resp => {
                    if (resp) {
                        this.activeName = -1;
                    }
                })
            },
            cancelUpdate() {
                this.activeName = -1;
            },
            addRole() {
                if (this.role.name === '' || this.role.nameZh === '') {
                    this.$message.error('数据不能为空');
                    return;
                }
                this.postRequest("/system/basic/permiss/", this.role).then(resp => {
                    if (resp) {
                        this.initRoles();
                        this.role.nameZh = '';
                        this.role.name = '';
                    }
                })
            },
            deleteRole(item){
                this.$confirm('此操作将永久删除所'+item.nameZh+', 是否继续?', '提示',{
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/basic/permiss/"+item.id).then(resp => {
                        if (resp) {
                            this.initRoles();
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

</style>