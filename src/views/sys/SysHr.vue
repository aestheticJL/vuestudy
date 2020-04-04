<template>
    <div>
        <div style="margin-top: 13px;display:flex;justify-content: center">
            <el-input v-model="keyword" placeholder="通过用户名搜索" prefix-icon="el-icon-search"
                      style="width: 400px;margin-right: 13px" @keydown.enter.native="searchHr"/>
            <el-button icon="el-icon-search" type="primary" @click="searchHr">搜索</el-button>
        </div>
        <div class="hr-container">
            <el-card class="hrCard" v-for="(hr,index) in hrs" :key="index">
                <div slot="header" class="clearfix">
                    <span>{{hr.name}}</span>
                    <el-button style="float: right;padding: 3px 0;color: red" type="text" icon="el-icon-delete" @click="deleteHr(hr)"/>
                </div>
                <div>
                    <div style="display: flex;justify-content: center">
                        <el-avatar :size="75" :src="hr.userface"/>
                    </div>
                    <div class="information">
                        <div>用户名：{{hr.name}}</div>
                        <div>手机号码：{{hr.phone}}</div>
                        <div>电话号码：{{hr.telephone}}</div>
                        <div>地址：{{hr.address}}</div>
                        <div>用户状态：
                            <el-switch
                                    v-model="hr.enabled"
                                    active-text="启用"
                                    inactive-text="禁用"
                                    active-color="#13ce66"
                                    inactive-color="#ff4949"
                                    @change="updateHr(hr)">
                            </el-switch>
                        </div>
                        <div>用户角色：
                            <el-tag type="success" v-for="(role,index) in hr.roles" :key="index"
                                    style="margin-right: 4px">
                                {{role.nameZh}}
                            </el-tag>
                            <el-popover
                                    placement="right"
                                    title="角色列表"
                                    width="200"
                                    trigger="click"
                                    @show="showRoles(hr.roles)"
                                    @hide="hideRoles(hr)">
                                <el-select v-model="selectedRoles" placeholder="请选择" multiple>
                                    <el-option
                                            v-for="(role,index) in allRoles"
                                            :key="index"
                                            :label="role.nameZh"
                                            :value="role.id">
                                    </el-option>
                                </el-select>
                                <el-button slot="reference" type="text" icon="el-icon-more" @click=""/>
                            </el-popover>
                        </div>
                        <div>备注：{{hr.remark}}</div>
                    </div>
                </div>
            </el-card>
        </div>
    </div>
</template>

<script>
    export default {
        name: "SysHr",
        data() {
            return {
                keyword: '',
                hrs: [],
                allRoles: [],
                selectedRoles: []
            }
        },
        mounted() {
            this.initHrs();
        },
        methods: {
            initHrs() {
                this.getRequest("/system/hr/?keyword=" + this.keyword).then(resp => {
                    if (resp) {
                        this.hrs = resp;
                    }
                })
            },
            searchHr() {
                this.initHrs();
            },
            deleteHr(hr){
                this.$confirm('此操作将永久删除所'+hr.name+'用户, 是否继续?', '提示',{
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.deleteRequest("/system/hr/"+hr.id).then(resp => {
                        if (resp) {
                            this.initHrs();
                        }
                    })
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            updateHr(hr) {
                delete hr.roles;
                this.putRequest("/system/hr/", hr).then(resp => {
                    if (resp) {
                        this.initHrs();
                    }
                })
            },
            initRoles() {
                this.getRequest("/system/hr/roles").then(resp => {
                    if (resp) {
                        this.allRoles = resp;
                    }
                })
            },
            showRoles(roles) {
                this.initRoles();
                this.selectedRoles = [];
                roles.forEach(r => {
                    this.selectedRoles.push(r.id);
                })
            },
            hideRoles(hr) {
                let roles = [];
                Object.assign(roles, hr.roles);
                let flag = false;
                if (roles.length !== this.selectedRoles.length) {
                    flag = true;
                } else {
                    for (let i = 0; i < roles.length; i++) {
                        let role = roles[i];
                        for (let j = 0; j < this.selectedRoles.length; j++) {
                            if (role.id === this.selectedRoles[j]) {
                                roles.splice(i, 1);
                                i--;
                                break;
                            }
                        }
                    }
                    if (roles.length !== 0) {
                        flag = true;
                    }
                }
                if (flag) {
                    let url = '/system/hr/role?hrid=' + hr.id;
                    this.selectedRoles.forEach(rid => {
                        url += '&rids=' + rid;
                    });
                    this.putRequest(url).then(resp => {
                        if (resp) {
                            this.initHrs();
                        }
                    })
                }
            }
        }
    }
</script>

<style>
    .hr-container {
        display: flex;
        flex-wrap: wrap;
    }

    .hrCard {
        width: 350px;
        margin: 46px 46px 46px 0;
    }

    .information {
        margin-top: 20px;
    }

    .information div {
        color: #2892ea;
        font-size: 13px;
    }
</style>