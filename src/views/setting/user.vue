<template>
  <div>
    <div class="zan-nav">
      <el-form label-position="right" label-width="84px" :inline="true" :model="queryParams" class="demo-form-inline">
        <el-form-item  label="用户姓名：" >
          <el-input
              size="medium"
              clearable
              @keyup.enter="getInfo"
              v-model.trim="queryParams.staffName"
              placeholder="请输入姓名"
          ></el-input>
        </el-form-item>
        <el-form-item label="用户ID：">
          <el-input
              size="medium"
              clearable
              @keyup.enter="getInfo"
              v-model.trim="queryParams.staffId"
              placeholder="请输入ID"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button icon="el-icon-search" size="small" type="primary" @click="getInfo">查 询</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="zan-table">
      <el-table
          :data="userList"
          height="calc(100vh - 320px)"
          style="width: 100%">
        <el-table-column
            prop="staffName"
            label="用户姓名"
        >
        </el-table-column>
        <el-table-column
            prop="staffId"
            label="用户ID"
        >
        </el-table-column>
        <el-table-column
            prop="username"
            label="用户名"
        >
        </el-table-column>
        <el-table-column
            prop="sex"
            label="性别">
          <template  #default="scope">{{$filters.Gender(scope.row.sex)}}</template>
        </el-table-column>
        <el-table-column
            prop="phone"
            label="手机号码">
        </el-table-column>
        <el-table-column
            prop="address"
            label="用户住址">
        </el-table-column>
        <el-table-column
            prop="createTime"
            label="注册时间">
        </el-table-column>
        <el-table-column
            align="center"
            label="操作"
           >
          <template #default="scope">
            <el-space spacer="|" style="color:#dedede">
              <el-button type="text" @click="baseInfoEdit(scope.row)">编辑</el-button>
              <el-button type="text"
                         @click="deleteUser(scope.row)">删除
              </el-button>
            </el-space>
          </template>
        </el-table-column>
      </el-table>
      <component :is="components.pagination" class="zan-pagination" @change="getInfo"></component>
    </div>
    <base-info ref="baseInfoRef" v-model:baseVisible="baseVisible" @reFresh="reFresh"></base-info>
  </div>
</template>
<script>
import {defineComponent, ref, reactive, toRefs, onMounted, shallowRef, provide} from "vue";
import {getAll, delUser} from "../../api/user.js";
import {ElMessageBox} from 'element-plus';
import baseInfo from "../../components/Setting/baseInfo.vue";
import Pagination from "../../components/Pagination/index.vue";

export default defineComponent({
  name: "user",
  components: {
    Pagination,
    baseInfo
  },
  emits: [],
  setup() {
    const baseInfoRef = ref('null');
    const state = reactive({
      userList: [], //存储用户信息
      baseVisible: false,//基本信息弹框
      queryParams: {
        staffName: "",
        staffId: "",
      },
      pagination: {
        total: 1,
        page: 1
      }
    });

    provide("pagination", state.pagination); //父子传参

    const components = shallowRef({ //子组件注册
      pagination:Pagination
    })

    const getInfo = (val) => {  //查询
      let dataWare = {
        ...state.queryParams,
        pageSize: val && val.pageSize || 10,
        page: val && val.page || 1
      };
      getAll(dataWare).then((res) => {
        state.userList = res?.data?.list || []; //表格数据
        state.pagination.total = res?.data?.total || 1; //总条数
        state.pagination.page = res?.data?.page || 1; //当前页
      });
    };

    const reFresh = (val) => {
      //刷新
      if (val) getInfo();
    }


    const baseInfoEdit = (row) => {
      //打开编辑信息弹框
      baseInfoRef.value.openBaseInfo(row);
    }


    const deleteUser = (row) => {  //删除用户
      ElMessageBox.confirm('此操作将注销该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        delUser({
          id: row.id
        }).then(() => {
          getInfo();
        });

        }).catch(() => {

        });
    }

    onMounted(() => {
      getInfo();
    });

    return {
      ...toRefs(state),
      components,
      baseInfoRef,
      getInfo,
      deleteUser,
      baseInfoEdit,
      reFresh
    };
  },
});
</script>
