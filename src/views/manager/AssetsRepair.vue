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
		<el-table-column prop="assetsCode" label="资产编号"></el-table-column>
        <el-table-column prop="status" label="维修状态">
          <template v-slot="scope">
            <el-tag type="info" v-if="scope.row.status === '处理中'">处理中</el-tag>
            <el-tag type="primary" v-if="scope.row.status === '维修中'">维修中</el-tag>
            <el-tag type="success" v-if="scope.row.status === '已完成'">已完成</el-tag>
            <el-tag type="danger" v-if="scope.row.status === '拒绝'">拒绝</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="reason" label="报修理由"></el-table-column>
        <el-table-column prop="date" label="报修日期"></el-table-column>
        <el-table-column prop="staffName" label="报修人"></el-table-column>
        <el-table-column label="审核" align="center" width="180" v-if="user.role === 'ADMIN'">
          <template v-slot="scope">
            <el-button v-if="scope.row.status === '处理中'" size="mini" type="success" plain @click="changeStatus(scope.row, '维修中')">通过</el-button>
            <el-button v-if="scope.row.status === '处理中'" size="mini" type="danger" plain @click="changeStatus(scope.row, '拒绝')">拒绝</el-button>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center" width="200">
          <template v-slot="scope">
            <el-button size="mini" type="success" plain @click="changeStatus(scope.row, '已完成')" v-if="scope.row.status === '维修中'">完成维修</el-button>
            <el-button size="mini" type="danger" plain @click="del(scope.row.id)" v-if="scope.row.status === '处理中'">删除</el-button>
            <el-button size="mini" type="info" plain @click="viewDetail(scope.row)" v-if="scope.row.status !== '处理中'">查看详情</el-button>
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

    <!-- 维修详情对话框 -->
    <el-dialog title="维修详情" :visible.sync="detailVisible" width="50%" :close-on-click-modal="false">
      <el-descriptions :column="2" border>
        <el-descriptions-item label="资产名称">{{ detailForm.assetsName }}</el-descriptions-item>
        <el-descriptions-item label="资产编号">{{ detailForm.assetsCode }}</el-descriptions-item>
        <el-descriptions-item label="维修状态">
          <el-tag :type="getStatusTagType(detailForm.status)">{{ detailForm.status }}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="报修人">{{ detailForm.staffName }}</el-descriptions-item>
        <el-descriptions-item label="报修日期">{{ detailForm.date }}</el-descriptions-item>
        <el-descriptions-item label="报修理由">{{ detailForm.reason }}</el-descriptions-item>
        <el-descriptions-item label="故障描述" :span="2">{{ detailForm.faultDesc }}</el-descriptions-item>
        <el-descriptions-item label="维修结果" :span="2" v-if="detailForm.repairResult">{{ detailForm.repairResult }}</el-descriptions-item>
        <el-descriptions-item label="完成时间" v-if="detailForm.completeTime">{{ detailForm.completeTime }}</el-descriptions-item>
      </el-descriptions>
      
      <div slot="footer" class="dialog-footer">
        <el-button @click="detailVisible = false">关闭</el-button>
        <el-button v-if="detailForm.status === '维修中'" type="primary" @click="completeRepair">完成维修</el-button>
      </div>
    </el-dialog>

  </div>
</template>
<script>
export default {
  name: "AssetsRepair",
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
      },
      ids: [],
      detailVisible: false,
      detailForm: {}
    }
  },
  created() {
    this.load(1)
  },
  methods: {
    changeStatus(row, status) {
      this.form = JSON.parse(JSON.stringify(row))
      this.form.status = status
      this.$confirm('您确定操作吗？', '确认审核', {type: "warning"}).then(response => {
        this.$request.put('/assetsRepair/update', this.form).then(res => {
          if (res.code === '200') {
            this.$message.success('审核成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(e => {})
    },
    handleAdd() {   // 新增数据
      this.form = {}  // 新增数据的时候清空数据
      this.fromVisible = true   // 打开弹窗
    },
    handleEdit(row) {   // 编辑数据
      this.form = JSON.parse(JSON.stringify(row))  // 给form对象赋值  注意要深拷贝数据
      this.fromVisible = true   // 打开弹窗
    },
    save() {   // 保存按钮触发的逻辑  它会触发新增或者更新
      this.$refs.formRef.validate((valid) => {
        if (valid) {
          this.$request({
            url: this.form.id ? '/assetsRepair/update' : '/assetsRepair/add',
            method: this.form.id ? 'PUT' : 'POST',
            data: this.form
          }).then(res => {
            if (res.code === '200') {  // 表示成功保存
              this.$message.success('保存成功')
              this.load(1)
              this.fromVisible = false
            } else {
              this.$message.error(res.msg)  // 弹出错误的信息
            }
          })
        }
      })
    },
    del(id) {   // 单个删除
      this.$confirm('您确定删除吗？', '确认删除', {type: "warning"}).then(response => {
        this.$request.delete('/assetsRepair/delete/' + id).then(res => {
          if (res.code === '200') {   // 表示操作成功
            this.$message.success('操作成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(() => {
      })
    },
    handleSelectionChange(rows) {   // 当前选中的所有的行数据
      this.ids = rows.map(v => v.id)
    },
    delBatch() {   // 批量删除
      if (!this.ids.length) {
        this.$message.warning('请选择数据')
        return
      }
      this.$confirm('您确定批量删除这些数据吗？', '确认删除', {type: "warning"}).then(response => {
        this.$request.delete('/assetsRepair/delete/batch', {data: this.ids}).then(res => {
          if (res.code === '200') {   // 表示操作成功
            this.$message.success('操作成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(() => {
      })
    },
    load(pageNum) {  // 分页查询
      if (pageNum) this.pageNum = pageNum
      this.$request.get('/assetsRepair/selectPage', {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          assetsName: this.assetsName,
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
    
    viewDetail(row) {
      this.detailForm = JSON.parse(JSON.stringify(row))
      this.detailVisible = true
    },
    
    completeRepair() {
      this.$confirm('确认完成维修？', '确认操作', {type: "warning"}).then(response => {
        this.detailForm.status = '已完成'
        this.detailForm.repairResult = '维修完成'
        this.detailForm.completeTime = new Date().toLocaleString()
        
        this.$request.put('/assetsRepair/update', this.detailForm).then(res => {
          if (res.code === '200') {
            this.$message.success('维修完成')
            this.detailVisible = false
            this.load(1)
          } else {
            this.$message.error(res.msg)
          }
        })
      }).catch(e => {})
    },
    
    getStatusTagType(status) {
      switch(status) {
        case '处理中': return 'info';
        case '维修中': return 'primary';
        case '已完成': return 'success';
        case '拒绝': return 'danger';
        default: return '';
      }
    }
  }
}
</script>

<style scoped>

</style>