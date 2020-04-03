<template>
    <div style="width: 500px">
        <div>
            <el-input
                    placeholder="输入部门名称进行搜索"
                    v-model="filterText"
                    prefix-icon="el-icon-search"
                    style="width: 300px;margin-right: 13px">
            </el-input>
        </div>
        <el-tree
                class="filter-tree"
                :data="deps"
                :props="defaultProps"
                :filter-node-method="filterNode"
                :expand-on-click-node=false
                ref="tree">
             <span class="custom-tree-node" slot-scope="{ node, data }"
                   style="display: flex;justify-content:space-between;width: 100%">
        <span>{{ node.label }}</span>
        <span>
          <el-button
                  class="depBtn"
                  type="primary"
                  size="mini"
                  @click="() => showAddDepView(data)">
            添加部门
          </el-button>
          <el-button
                  class="depBtn"
                  type="danger"
                  size="mini"
                  @click="() => deleteDep(data)">
            删除部门
          </el-button>
        </span>
      </span>
        </el-tree>
        <el-dialog
                title="添加部门"
                :visible.sync="dialogVisible"
                width="30%">
            <table>
                <tr>
                    <td>
                        <el-tag>上级部门</el-tag>
                    </td>
                    <td>{{pName}}</td>
                </tr>
                <tr>
                    <td>
                        <el-tag>部门名称</el-tag>
                    </td>
                    <td>
                        <el-input v-model="dep.name" placeholder="请输入部门名称" size="mini"/>
                    </td>
                </tr>
            </table>
            <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="doAddDep">确 定</el-button>
  </span>
        </el-dialog>

    </div>
</template>

<script>
    export default {
        watch: {
            filterText(val) {
                this.$refs.tree.filter(val);
            }
        },
        name: "DepMana",
        data() {
            return {
                filterText: '',
                deps: [],
                dep: {
                    name: '',
                    parentId: -1
                },
                pName: '',
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                dialogVisible: false
            }
        },
        mounted() {
            this.initDep();
        },
        methods: {
            filterNode(value, data) {
                if (!value) return true;
                return data.name.indexOf(value) !== -1;
            },
            initDep() {
                this.getRequest("/system/basic/department/").then(resp => {
                    if (resp) {
                        this.deps = resp;
                    }
                })
            },
            showAddDepView(data) {
                this.dialogVisible = true;
                this.pName = data.name;
                this.dep.parentId = data.id;
            },
            deleteDep(data) {
                this.$confirm('此操作将永久删除所'+data.name+'部门, 是否继续?', '提示',{
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/basic/department/"+data.id).then(resp => {
                        if (resp) {
                            this.removeDepFormDeps(this.deps,data);
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            removeDepFormDeps(deps,dep){
                for (let i = 0; i < deps.length; i++) {
                    let d = deps[i];
                    if (d.id === dep.id) {
                        deps.splice(i, 1);
                        return;
                    } else {
                        this.removeDepFormDeps(d.children, dep);
                    }
                }
            },
            replaceDep() {
                this.dep.parentId = -1;
                this.dep.name = '';
                this.pName = '';
            },
            addDepToDeps(deps, dep) {
                for (let i = 0; i < deps.length; i++) {
                    let d = deps[i];
                    if (d.id === dep.parentId) {
                        d.children = d.children.concat(dep);
                        return;
                    } else {
                        this.addDepToDeps(d.children, dep);
                    }
                }
            },
            doAddDep() {
                this.postRequest("/system/basic/department/", this.dep).then(resp => {
                    if (resp) {
                        this.addDepToDeps(this.deps,resp.obj);
                        this.dialogVisible = false;
                        this.replaceDep();
                    }
                })
            }
        }
    }
</script>

<style>
    .depBtn {
        padding: 2px;
    }
</style>