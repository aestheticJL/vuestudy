<template>
    <div>
        <el-form :rules="rules" :model="loginForm" class="loginContainer" ref="loginForm">
            <el-form-item>
                <h1 class="loginTitle">
                    系统登录
                </h1>
            </el-form-item>
            <el-form-item prop="username">
                <el-input type="text" v-model="loginForm.username" auto-complete="off"
                          placeholder="请输入用户名"/>
            </el-form-item>
            <el-form-item prop="password">
                <el-input type="password" v-model="loginForm.password" auto-complete="off"
                          placeholder="请输入密码"/>
            </el-form-item>
            <el-checkbox v-model="checked" class="rememberMe">记住我</el-checkbox>
            <el-button type="primary" style="width: 100%" @click="submitLogin">登录</el-button>
        </el-form>
    </div>
</template>

<script>
    import {postKeyValueRequest} from "../utils/api";

    export default {
        name: "Login",
        data() {
            return {
                loginForm: {
                    username: '',
                    password: ''
                },
                rules: {
                    username: [{required: true, message: '请输入用户名', trigger: 'blur'}],
                    password: [{required: true, message: '请输入密码', trigger: 'blur'}]
                },
                checked: true
            }
        },
        methods: {
            submitLogin() {
                this.$refs.loginForm.validate((valid) => {
                    if (valid) {
                        postKeyValueRequest('/doLogin', this.loginForm).then(resp => {
                            if (resp) {
                                alert(JSON.stringify(resp))
                            }
                        })
                    } else {
                        this.$message.error('请输入用户名和密码');
                        return false;
                    }
                });
            }
        }
    }
</script>

<style>
    .loginContainer {
        border-radius: 15px;
        background-clip: padding-box;
        margin: 180px auto;
        width: 350px;
        padding: 15px 25px 35px 25px;
        background: #ffffff;
        border: 1px solid #eaeaea;
        box-shadow: 0 0 25px #cac6c6;
    }

    .loginTitle {
        margin: 0 auto;
        text-align: center;
        color: #505458;
    }

    .rememberMe {
        margin: 10px 0 10px 0;
    }
</style>