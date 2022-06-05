<template>
  <div class="pageAdimin">
    <el-card>
      <p class="driver_p0">
        <i style="color: #ffc107" class="el-icon-s-opportunity"></i
        >本页面只有admin权限下显示
      </p>
      <!--      <div class="driver_p0">你的权限页面是:</div>-->
      <!--      <el-tag v-for="item in roles" :key="item">{{ item }}</el-tag>-->
    </el-card>
    <div class="className">
      <el-card class="anoCard">
        <div slot="header">
          <span>用户管理</span>
        </div>
        <div class="searchDiv">
          <el-input
            type="text"
            placeholder="请输入用户名"
            class="width1"
            v-model="sch_order"
          ></el-input>
          <el-select
            v-model="sch_status"
            clearable
            class="width1"
            placeholde="请选择身份"
          >
            <el-option
              v-for="item in optionsl"
              :label="item.label"
              :value="item.value"
              :key="item.value"
            ></el-option>
          </el-select>
          <el-button type="primary" icon="el-icon-search" @click="searchTab()"
            >搜索</el-button
          >
          <el-button
            type="primary"
            icon="el-icon-circle-plus-outline"
            @click="addTab"
            >添加</el-button
          >
        </div>
        <el-table :data="tableData" border stripe>
          <el-table-column prop="id" label="序号" width="60"></el-table-column>
          <el-table-column label="用户名">小陈</el-table-column>
          <el-table-column
            prop="address"
            label="配送地址"
            width="210"
          ></el-table-column>
          <el-table-column prop="phone" label="联系电话"></el-table-column>
          <el-table-column prop="status" label="角色" width="90">
            <template slot-scope="scope">
              <el-tag :type="scope.row.status | tagClass">{{
                scope.row.status | statusText
              }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="300">
            <template slot-scope="scope">
              <el-button
                type="primary"
                @click="editTable(scope.$index, scope.row)"
                :disabled="scope.row.status !== 0 ? false : true"
                >编辑</el-button
              >
              <el-button
                type="danger"
                @click="toDelete(scope.row)"
                :disabled="scope.row.status !== 0 ? false : true"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          background
          layout="total, sizes, prev, pager, next"
          :page-sizes="pageSizes"
          :page-size="pageSize"
          :current-page="currentPage"
          :total="total"
          class="fyDiv"
          @size-change="handleSize"
          @current-change="handlePage"
        >
        </el-pagination>
      </el-card>
      <el-dialog title="用户" :visible.sync="diaIsShow" class="diaForm">
        <el-form
          ref="diaForm"
          :model="formData"
          :rules="rules"
          label-width="140px"
        >
          <el-form-item label="用户名">
            <el-input type="text" v-model="formData.id"></el-input>
          </el-form-item>
          <el-form-item label="配送地址" prop="address">
            <el-input
              type="text"
              placeholder="请输入地址"
              v-model="formData.address"
            ></el-input>
          </el-form-item>
          <el-form-item label="联系电话" prop="phone">
            <el-input
              type="text"
              placeholder="请输入电话"
              v-model="formData.phone"
            ></el-input>
          </el-form-item>
          <el-form-item label="角色" prop="status">
            <el-select
              v-model="formData.status"
              placeholde="请选择新增用户身份"
            >
              <el-option
                v-for="item in optionsl"
                :label="item.label"
                :value="item.value"
                :key="item.value"
                :disabled="item.value !== 0 ? false : true"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="changeTab('diaForm', editType)"
              >确认</el-button
            >
            <el-button @click="diaIsShow = false">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { getPageTab2 } from '@/api/table'
export default {
  data: function() {
    return {
      tableData: [],
      allList: [],
      schArr: [],
      sch_order: '',
      sch_status: null,
      sch_date: null,
      currentPage: 1,
      pageSize: 10,
      total: 0,
      pageSizes: [10, 20, 30, 40],
      diaIsShow: false,
      formData: {},
      editType: '',
      options: [
        { label: '管理员', value: 0 },
        { label: '用户', value: 1 },
        { label: '用户', value: 2 },
        { label: '用户', value: 3 }
      ],
      optionsl: [{ label: '管理员', value: 0 }, { label: '用户', value: 1 }],
      rowIndex: 0,
      rules: {
        // order: [{ required: true, message: '请输入订单号', trigger: 'blur' }],
        time: [
          {
            // type: 'datetime',
            required: true,
            message: '请输入时间',
            trigger: 'change'
          }
        ],
        address: [{ required: true, message: '请输入地址', trigger: 'blur' }],
        phone: [{ required: true, message: '请输入联系方式', trigger: 'blur' }],
        name: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
        status: [
          { required: true, message: '请选择角色身份', trigger: 'change' }
        ]
      }
    }
  },
  created() {
    this._getPageTab2()
  },
  filters: {
    statusText(val) {
      if (val === undefined) return
      if (val === 0) {
        return '管理员'
      } else {
        return '用户'
      }
    },
    tagClass(val) {
      if (val === undefined) return
      if (val === 0) {
        return 'success'
      } else {
        return 'info'
      }
    }
  },
  methods: {
    handleSize(val) {
      this.pageSize = val
      this.getPageData()
    },
    handlePage(val) {
      this.currentPage = val
      this.getPageData()
    },
    _getPageTab2() {
      getPageTab2()
        .then(res => {
          this.allList = res.data.tableList
          this.schArr = this.allList
          this.getPageData()
          this.total = res.data.total
        })
        .catch(error => {
          this.$message.error(error.message)
        })
    },
    getPageData() {
      let start = (this.currentPage - 1) * this.pageSize
      let end = start + this.pageSize
      this.tableData = this.schArr.slice(start, end)
    },
    // 查找
    searchTab() {
      let arrList = []
      for (let item of this.allList) {
        if (
          this.sch_order.trim() === '' &&
          this.sch_status === null &&
          this.sch_date === null
        ) {
          arrList = this.allList
          break
        } else if (
          item.order.startsWith(this.sch_order) &&
          (this.sch_status !== null ? item.status === this.sch_status : true) &&
          (this.sch_date !== null ? item.time.startsWith(this.sch_date) : true)
        ) {
          let obj = Object.assign({}, item)
          arrList.push(obj)
        }
      }
      this.schArr = arrList
      this.total = arrList.length
      this.currentPage = 1
      this.pageSize = 10
      this.getPageData()
    },
    // add
    addTab() {
      this.formData = {}
      this.diaIsShow = true
      this.formData.order = (Math.random() * 10e18).toString()
      this.formData.id = this.allList.length + 1
      this.editType = 'add'
      this.$nextTick(() => {
        this.$refs.diaForm.clearValidate()
      })
    },
    toDelete(row) {
      row.status = 3
      this.$notify({
        title: '成功',
        message: '已取消该订单',
        type: 'success'
      })
    },
    // 编辑
    editTable(index, row) {
      this.formData = Object.assign({}, row)
      this.editType = 'update'
      this.diaIsShow = true
      this.$nextTick(() => {
        this.$refs.diaForm.clearValidate()
      })
      this.rowIndex = index
    },
    changeTab(form, type) {
      this.$refs[form].validate(valid => {
        if (valid) {
          if (type === 'update') {
            // 改变整个表格数据
            let start = (this.currentPage - 1) * this.pageSize
            this.allList[start + this.rowIndex] = Object.assign(
              {},
              this.formData
            )
            // 解决数组不能通过索引响应数据变化
            this.$set(
              this.tableData,
              this.rowIndex,
              Object.assign({}, this.formData)
            )
            this.$notify({
              title: '成功',
              message: '订单已修改成功',
              type: 'success'
            })
          } else {
            this.tableData.unshift(Object.assign({}, this.formData))
            this.allList.push(Object.assign({}, this.formData))
          }
          this.diaIsShow = false
        } else {
          return
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.fyDiv {
  float: right;
  margin-top: 30px;
  padding-bottom: 20px;
}
.anoCard .el-table .el-button {
  padding: 8px 18px;
  font-size: 12px;
}
.searchDiv {
  margin-bottom: 20px;
  .el-button {
    padding: 11px 20px;
  }
}
.width1 {
  width: 180px;
  margin-right: 10px;
}
.diaForm {
  .el-input {
    width: 350px;
  }
}
</style>
<style lang="scss">
.anoCard {
  .el-card__body:after {
    content: '';
    clear: both;
    width: 0;
    height: 0;
    visibility: hidden;
    display: block;
  }
}
.diaForm .el-form-item__label {
  padding-right: 20px;
}
.searchDiv [class^='el-icon'] {
  color: #fff;
}
</style>
