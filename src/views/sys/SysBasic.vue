<template>
        <el-tabs v-model="activeName" type="card">
            <el-tab-pane label="部门管理" name="depMana"><DepMana/></el-tab-pane>
            <el-tab-pane label="职位管理" name="posMana"><PosMana/></el-tab-pane>
            <el-tab-pane label="职称管理" name="jobLevelMana"><JobLevelMana/></el-tab-pane>
            <el-tab-pane label="奖惩规则" name="ecMana"><EcMana/></el-tab-pane>
            <el-tab-pane label="权限组" name="permissMana"><PermissMana/></el-tab-pane>
        </el-tabs>
</template>

<script>
    import DepMana from '../../components/sys/basic/DepMana'
    import EcMana from '../../components/sys/basic/EcMana'
    import JobLevelMana from '../../components/sys/basic/JobLevelMana'
    import PosMana from '../../components/sys/basic/PosMana'
    import PermissMana from '../../components/sys/basic/PermissMana'
    export default {
        name: "SysBasic",
        data(){
            return{
                activeName:'depMana'
            }
        },
        components:{
            DepMana,
            EcMana,
            JobLevelMana,
            PermissMana,
            PosMana
        }
    }
</script>

<style scoped>
</style>