<!-- src/views/account/Admin.vue -->
<script setup>
  import { ref, onMounted, computed } from 'vue';
  import { getAdminList, deleteAdmin, addAdmin, updateAdmin } from '@/api/admin'
  import { constantRoutes, asyncRoutes } from '@/router'
  import { ElTree } from 'element-plus'
  import { getRoutes } from '@/utils/get-routes'


  // 将展示数据adminList 变为了 计算属性的数据
  // 将依据原始数据 originalList 以及 页码currentPage  和每页显示个数pageSize 一起计算出来显示的数据adminList
 
  const currentPage = ref(1)
  const pageSize = ref(10)
  const originalList = ref([])
  const changeCurrentPage = (val) => { // 页码改变函数
    currentPage.value = val
  }
  const changeSize = (val) => { // 每页显示个数改变的函数
    pageSize.value = val
  }

  const adminList = computed(() => {
    // 截取数组的某一些项
    // 深拷贝数据作为备份
    // const arr = JSON.parse(JSON.stringify(originalList.value))
    // const newArr = arr.splice((currentPage.value - 1) * pageSize.value, pageSize.value)
    const newArr = originalList.value.filter((item, index) => {
      return index >= (currentPage.value - 1) * pageSize.value && index < (currentPage.value - 1) * pageSize.value + pageSize.value
    })
    // console.log(originalList.value)
    return newArr
  })
  const getAdminListData = () => {
    getAdminList().then(res => {
      console.log('admin', res.data)
      originalList.value = res.data.data
    })
  }
  onMounted(() => {
    getAdminListData()
  })

  const removeItem = (adminid) => {
    deleteAdmin({ adminid }).then(() => {
      getAdminListData()
    })
  }

  // 添加管理员
  const drawer = ref(false)
  const adminname = ref('')
  const password = ref('')
  const role = ref(1)
  const checkedKeys = ref([])
  const menuList = [...constantRoutes, ...asyncRoutes]
  const treeData = getRoutes(menuList)
  const onTreeCheck = (_, obj) => {
    console.log("aaaa",obj)
    checkedKeys.value = [...obj.halfCheckedKeys, ...obj.checkedKeys].sort()
  }
  const treeAddRef = ref(null); // 在setup函数中添加ref引用
  const addAdminFn = () => {
    addAdmin({
      adminname: adminname.value,
      password: password.value,
      role: role.value,
      checkedKeys: checkedKeys.value
    }).then(() => {
      // 重置表单状态
    adminname.value = ''
    password.value = ''
    role.value = 1
    checkedKeys.value= []
    // 关闭抽屉效果
    drawer.value = false
      getAdminListData()
       // 重置树形控件的选中状态
       treeAddRef.value.setCheckedKeys([])
    })
  }
  const closeDrawer = () => {
    // 重置表单状态
    adminname.value = ''
    password.value = ''
    role.value = 1
    checkedKeys.value = []
    // 关闭抽屉效果
    drawer.value = false
  }

  // 修改管理员
  const treeUpdateRef = ref()
  const dialog = ref(false)
  const updateAdminInfo = (item) => {
    dialog.value = true
    adminname.value = item.adminname
    role.value = item.role
    // console.log(item)
    // checkedKeys.value = item.checkedKeys
    // 添加延时器确保DOM加载完毕， 否则显示 setCheckedKeys 未定义
    setTimeout(() => {
      treeUpdateRef.value.setCheckedKeys(item.checkedKeys, false)
    }, 0)
  }
  const updateAdminInfoFn = () => {
    // console.log('111', {
    //   adminname: adminname.value,
    //   role: role.value,
    //   checkedKeys: checkedKeys.value
    // })
    updateAdmin({
      adminname: adminname.value,
      role: role.value,
      checkedKeys: checkedKeys.value
    }).then((res) => {
      // console.log('22222', res.data)
      // 重置表单状态
      adminname.value = ''
      role.value = 1
      checkedKeys.value = []
      // 关闭抽屉效果
      dialog.value = false
      getAdminListData()
    })
  }
</script>

<template>
  <el-button type="primary" @click="drawer=true">添加管理员</el-button>
  <el-table :data="adminList" style="width: 100%">
    <el-table-column label="序号">
      <template #default="scope">
        <!-- scope.$index 代表索引值 -->
        <span>{{ (currentPage - 1) * pageSize + scope.$index + 1 }}</span>
      </template>
    </el-table-column>
    <el-table-column prop="adminname" label="管理员账户" width="180" />
    <el-table-column prop="role" label="角色" width="180" >
      <template #default="scope">
        <!-- scope.row 代表一条数据 -->
        <el-tag v-if="scope.row.role === 2" class="ml-2" type="success">超级管理员</el-tag>
        <el-tag v-else class="ml-2" type="info">管理员</el-tag>
      </template>
    </el-table-column>
    <el-table-column label="操作">
      <template #default="scope">
        <el-button size="small" @click="updateAdminInfo(scope.row)">编辑</el-button>
        <el-popconfirm title="确定删除吗?" @confirm="removeItem(scope.row.adminid)">
          <template #reference>
            <el-button
              size="small"
              type="danger"
              >删除</el-button
            >
          </template>
        </el-popconfirm>
        
      </template>
    </el-table-column>
  </el-table>
  <el-pagination 
    background 
    layout="sizes, prev, pager, next" 
    :page-sizes="[5, 10, 20, 30]"
    :total="originalList.length" 
    @current-change="changeCurrentPage"
    @size-change="changeSize"
    />
    <!-- 添加管理员  抽屉效果 -->
    <el-drawer v-model="drawer" title="I am the title" :with-header="false" @close="closeDrawer">
      <h1>添加管理员</h1>
      <el-input v-model="adminname" placeholder="管理员账户" clearable/>
      <el-input v-model="password" placeholder="密码" clearable />
      <el-select v-model="role" placeholder="管理员角色" >
        <el-option
          label="管理员"
          :value="1"
        />
        <el-option
          label="超级管理员"
          :value="2"
        />
      </el-select>
      <el-tree
        ref="treeAddRef"
        :data="treeData"
        show-checkbox
        node-key="id"
        @check="onTreeCheck"
      />
      <el-button type="primary" @click="addAdminFn">添加</el-button>
    </el-drawer>
    <!-- 编辑管理员 -->
    <el-dialog v-model="dialog" title="编辑管理员信息">
      <el-input v-model="adminname" readonly placeholder="管理员账户" clearable/>
      <el-select v-model="role" placeholder="管理员角色" >
        <el-option
          label="管理员"
          :value="1"
        />
        <el-option
          label="超级管理员"
          :value="2"
        />
      </el-select>
      <el-tree
        ref="treeUpdateRef"
        :data="treeData"
        show-checkbox
        node-key="id"
        @check="onTreeCheck"
      />
      <template #footer>
        <span class="dialog-footer">
          <el-button type="primary" @click="updateAdminInfoFn">
            修改
          </el-button>
        </span>
      </template>
    </el-dialog>
</template>