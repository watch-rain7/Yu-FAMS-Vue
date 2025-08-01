<template>
  <div>
    <div class="search">
      <el-input placeholder="请输入资产名称查询" style="width: 200px" v-model="assetsName"></el-input>
      <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
      <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
    </div>

    <div class="table">
      <el-table :data="tableData" strip @selection-change="handleSelectionChange">
        <el-table-column prop="id" label="ID" width="70" align="center" sortable></el-table-column>
        <el-table-column prop="assetsName" label="资产名称"></el-table-column>
        <el-table-column prop="assetsNo" label="资产编号"></el-table-column>
        <el-table-column prop="staffName" label="员工名称"></el-table-column>
        <el-table-column prop="date" label="领用时间"></el-table-column>
        <el-table-column prop="returnDate" label="归还日期">
          <template v-slot="scope">
            <strong style="color: red" v-if="(scope.row.returnStatus === '未归还')  &&
             (new Date(scope.row.returnDate).getTime() < new Date().getTime())">{{scope.row.returnDate}}</strong>
            <span v-else>{{scope.row.returnDate}}</span>
          </template>
        </el-table-column>
        <el-table-column prop="returnStatus" label="归还状态">
          <template v-slot="scope">
            <span style="color: #000" v-if="scope.row.returnStatus === '未归还'">未归还</span>
            <span style="color: #049f58" v-if="scope.row.returnStatus === '已归还'">已归还</span>
          </template>
        </el-table-column>

        <el-table-column label="操作" align="center" width="160">
          <template v-slot="scope">
            <el-button size="mini" type="primary" plain @click="changeStatus(scope.row,'已归还')">归还</el-button>
            <el-button size="mini" type="warning" plain @click="handleRepair(scope.row)">维修</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
            background
            @current-change="handleCurrentChange"
            :current-page="pageNum"
            :page-sizes="[5, 10, 20]"
            :page-size="pageSize"
            layout="total, prev, pager, next"
            :total="total">
        </el-pagination>
      </div>
    </div>

    <el-dialog title="资产报修" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
        <el-form-item label="资产名称">
          <el-input v-model="form.assetsName" disabled></el-input>
        </el-form-item>
        <el-form-item label="资产编号">
          <el-input v-model="form.assetsCode" disabled></el-input>
        </el-form-item>
        <el-form-item label="报修理由" prop="reason">
          <el-input type="textarea" :rows="3" v-model="form.reason" placeholder="请描述报修理由"></el-input>
        </el-form-item>
        <el-form-item label="故障描述" prop="faultDesc">
          <el-input type="textarea" :rows="3" v-model="form.faultDesc" placeholder="请详细描述故障情况"></el-input>
        </el-form-item>
      </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="fromVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveRepair">确 定</el-button>
        </div>
      </el-dialog>

  </div>
</template>
<script>


export default {
  assetsName: "StaffAssets",
  data() {
    return {
      tableData: [],  // 所有的数据
      pageNum: 1,   // 当前的页码
      pageSize: 10,  // 每页显示的个数
      total: 0,
      assetsName: null,
      fromVisible: false,
      form: {},
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      rules: {
        reason: [
          { required: true, message: '请输入报修理由', trigger: 'blur' }
        ],
        faultDesc: [
          { required: true, message: '请输入故障描述', trigger: 'blur' }
        ]
      },
      ids: []
    }
  },
  created() {
    this.load(1)
  },
  methods: {
    saveRepair() {   // 保存按钮触发的逻辑  它会触发新增或者更新
      this.$refs.formRef.validate((valid) => {
        if (valid) {
          this.$request({
            url: '/assetsRepair/add',
            method: 'POST',
            data: this.form
          }).then(res => {
            if (res.code === '200') {  // 表示成功保存
              this.$message.success('报修申请提交成功')
              this.load(1)
              this.fromVisible = false
            } else {
              this.$message.error(res.msg)  // 弹出错误的信息
            }
          })
        }
      })
    },
    handleRepair(row){
      this.form = {
        receiveId: row.id,
        assetsName: row.assetsName,
        assetsCode: row.assetsNo,
        reason: '',
        faultDesc: ''
      }
      this.fromVisible = true
    },
    changeStatus(row, returnStatus) {
      this.form = JSON.parse(JSON.stringify(row))
      this.form.returnStatus = returnStatus
      this.$confirm('您确定归还资产吗？', '确认归还', {type: "warning"}).then(response => {
        this.$request.put('/assetsReceive/update', this.form).then(res => {
          if (res.code === '200') {
            this.$message.success('归还成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(e => {})
    },
    load(pageNum) {  // 分页查询
      if (pageNum) this.pageNum = pageNum
      this.$request.get('/assetsReceive/selectPage', {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          assetsName: this.assetsName,
          status:'通过',
          returnStatus:'未归还',
        }
      }).then(res => {
        if (res.code === '200') {
          this.tableData = res.data?.list
          this.total = res.data?.total
        } else {
          this.$message.error(res.msg)
        }
      })
    },
    reset() {
      this.assetsName = null
      this.load(1)
    },
    handleCurrentChange(pageNum) {
      this.load(pageNum)
    },
  }
}
</script>

<style scoped>

</style>