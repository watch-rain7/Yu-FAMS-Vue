<template>
  <div>
    <div class="search">
      <el-input placeholder="请输入资产名称查询" style="width: 200px" v-model="name"></el-input>
      <el-select placeholder="请选择资产分类" style="width: 200px; margin-left: 10px" v-model="category">
        <el-option v-for="item in categoryList" :key="item.id" :value="item.name" :label="item.name"></el-option>
      </el-select>
      <el-input placeholder="请输入资产编号查询" style="width: 200px; margin-left: 10px" v-model="no"></el-input>
      <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
      <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
    </div>

    <div class="operation" v-if="user.role=='ADMIN'">
      <el-button type="primary" plain @click="handleAdd">新增</el-button>
      <el-button type="danger" plain @click="delBatch">批量删除</el-button>
      <el-button type="info" plain @click="exportData">批量导出</el-button>
      <el-button type="success" plain @click="handleImport">批量导入</el-button>
    </div>

    <div class="table">
      <el-table :data="tableData" strip @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" align="center" v-if="user.role=='ADMIN'"></el-table-column>
        <el-table-column prop="id" label="序号" width="70" align="center" sortable></el-table-column>
        <el-table-column prop="name" label="资产名称"></el-table-column>
        <el-table-column prop="category" label="资产分类"></el-table-column>
        <el-table-column prop="no" label="资产编号"></el-table-column>
        <el-table-column prop="img" label="资产图片">
          <template v-slot="scope">
            <el-image v-if="scope.row.img" style="width: 50px" :src="scope.row.img" :preview-src-list="[scope.row.img]"></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="model" label="资产型号"></el-table-column>
        <el-table-column prop="num" label="资产剩余数量"></el-table-column>
        <el-table-column prop="date" label="购置日期"></el-table-column>
        <el-table-column prop="money" label="初始价值"></el-table-column>
        <el-table-column prop="depreciate" label="折旧方法"></el-table-column>
        <el-table-column prop="departmentName" label="使用部门"></el-table-column>
        <el-table-column prop="staffName" label="责任人"></el-table-column>
        <el-table-column prop="location" label="存放地点"></el-table-column>
        <el-table-column prop="status" label="状态"></el-table-column>
        <el-table-column prop="comment" label="备注"></el-table-column>
        <el-table-column label="操作" align="center" width="100" fixed="right" v-if="user.role=='STAFF'">
          <template v-slot="scope">
            <el-button size="mini" type="primary" plain @click="handleReceive(scope.row.id)">申请领用</el-button>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center" width="150" fixed="right" v-if="user.role=='ADMIN'">
          <template v-slot="scope">
            <el-button size="mini" type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" plain @click="del(scope.row.id)">删除</el-button>
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
    <el-dialog title="申请资产" :visible.sync="fromVisible1" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="assetsReceive" label-width="100px" style="padding-right: 50px" :rules="assetsReceiveRules" ref="assetsReceiveFormRef">
        <el-form-item label="归还日期" prop="returnDate">
          <el-date-picker format="yyyy-MM-dd" value-format="yyyy-MM-dd" placeholder="请选择归还日期"
                          v-model="assetsReceive.returnDate" style="width: 100%"></el-date-picker>
        </el-form-item>
        <el-form-item label="备注" prop="comment">
          <el-input type="textarea" :rows="5" v-model="assetsReceive.comment" placeholder="备注"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="fromVisible1 = false">取 消</el-button>
        <el-button type="primary" @click="saveAssetsReceive">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="资产信息" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
        <el-form-item label="资产名称" prop="name">
          <el-input v-model="form.name" placeholder="资产名称"></el-input>
        </el-form-item>
        <el-form-item label="资产分类" prop="category">
          <el-select style="width: 100%;" v-model="form.category">
            <el-option v-for="item in categoryList" :key="item.id" :value="item.name" :label="item.name"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="资产编号" prop="no">
          <el-input v-model="form.no" placeholder="资产编号"></el-input>
        </el-form-item>
        <el-form-item label="资产图片" prop="img">
          <el-upload
              :action="$baseUrl + '/files/upload'"
              :headers="{ token: user.token }"
              list-type="picture"
              :on-success="handleImgSuccess"
          >
            <el-button type="primary">上传</el-button>
          </el-upload>

        </el-form-item>
        <el-form-item label="资产型号" prop="model">
          <el-input v-model="form.model" placeholder="资产型号"></el-input>
        </el-form-item>
        <el-form-item label="数量" prop="num">
          <el-input-number :min="1" v-model="form.num" placeholder="数量"></el-input-number>
        </el-form-item>
        <el-form-item label="购置日期" prop="date">
          <el-date-picker format="yyyy-MM-dd" value-format="yyyy-MM-dd"
                          v-model="form.date" style="width: 100%"></el-date-picker>
        </el-form-item>
        <el-form-item label="初始价值" prop="money">
          <el-input v-model="form.money" placeholder="初始价值"></el-input>
        </el-form-item>
        <el-form-item label="折旧方法" prop="depreciate">
          <el-select style="width: 100%" v-model="form.depreciate">
            <el-option value="直线法"></el-option>
            <el-option value="工作量法"></el-option>
            <el-option value="双倍余额递减法"></el-option>
            <el-option value="年度总和法"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="使用部门ID" prop="departmentId">

          <el-select ref="selectTree" v-model="form.departmentId" clearable style="width: 100%;">
            <el-option
                v-for="item in departmentList"
                :key="item.id"
                :value="item.id"
                :label="item.name"
                style="display: none;"/>
            <el-tree
                :data="departmentTree"
                :props="{children: 'children', label: 'name'}"
                highlight-current
                @node-click="handleNodeClick"
                default-expand-all />
          </el-select>
        </el-form-item>
        <el-form-item label="责任人" prop="staffId">
            <el-select style="width: 100%" v-model="form.staffId">
              <el-option v-for="item in staffList" :key="item.id" :value="item.id" :label="item.name"></el-option>
            </el-select>
        </el-form-item>
        <el-form-item label="存放地点" prop="location">
          <el-input v-model="form.location" placeholder="存放地点"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select style="width: 100%" v-model="form.status">
            <el-option value="正常"></el-option>
            <el-option value="停用"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="备注" prop="comment">
          <el-input v-model="form.comment" placeholder="备注"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="fromVisible = false">取 消</el-button>
        <el-button type="primary" @click="save">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 批量导入对话框 -->
    <el-dialog title="批量导入资产" :visible.sync="importVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <div style="margin-bottom: 20px;">
        <el-alert
          title="导入说明"
          type="info"
          :closable="false"
          show-icon>
          <div slot="default">
            <p>1. 请下载模板文件，按照模板格式填写数据</p>
            <p>2. 支持Excel格式(.xlsx)文件</p>
            <p>3. 必填字段：资产名称、资产分类、资产编号</p>
            <p>4. 可选字段：资产型号、数量、购置日期、初始价值、折旧方法、存放地点、状态、备注</p>
          </div>
        </el-alert>
      </div>
      
      <div style="margin-bottom: 20px;">
        <el-button type="primary" @click="downloadTemplate">下载模板</el-button>
      </div>
      
      <el-upload
        class="upload-demo"
        :action="$baseUrl + '/assets/import'"
        :headers="{ token: user.token }"
        :on-success="handleImportSuccess"
        :on-error="handleImportError"
        :before-upload="beforeImport"
        accept=".xlsx,.xls"
        :show-file-list="true"
        :limit="1">
        <el-button type="success">选择文件</el-button>
        <div slot="tip" class="el-upload__tip">只能上传xlsx/xls文件</div>
      </el-upload>
      
      <div slot="footer" class="dialog-footer">
        <el-button @click="importVisible = false">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: "Assets",
  data() {
    return {
      tableData: [],  // 所有的数据
      pageNum: 1,   // 当前的页码
      pageSize: 10,  // 每页显示的个数
      total: 0,
      name: null,
      category:null,
      no:null,
      fromVisible: false,
      form: {},
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      rules: {
        name: [
          {required: true, message: '请输入名称', trigger: 'blur'}
        ],
        category: [
          {required: true, message: '请输入分类', trigger: 'blur'}
        ]
      },
      ids: [],
      staffList: [],
      departmentList:[],
      departmentTree:[],
      categoryList:[],
      assetsReceive: {},
      fromVisible1: false,
      assetsReceiveRules: {
        returnDate: [
          { required: true, message: '请选择归还日期', trigger: 'blur' }
        ],
      },
      importVisible: false
    }
  },
  created() {
    this.load(1)
    this.loadStaff()
    this.loadDepartment()
    this.loadCategory()
  },
  methods: {
    saveAssetsReceive() {
      this.$refs.assetsReceiveFormRef.validate((valid) => {
        if (valid) {
          this.$request.post('/assetsReceive/add', this.assetsReceive).then(res => {
            if (res.code === '200') {
              this.$message.success('申请成功')
              this.load(1)
              this.fromVisible1 = false
            } else {
              this.$message.error(res.msg)
            }
          })
        }
      })
    },
    handleReceive(assetsId){
      this.fromVisible1 = true
      this.assetsReceive = { assetsId: assetsId }
    },
    exportData(){
      window.open(this.$baseUrl+'/assets/export')
    },
    handleImport() {
      this.importVisible = true
    },
    downloadTemplate() {
      window.open(this.$baseUrl+'/assets/template')
    },
    beforeImport(file) {
      const isExcel = file.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' || 
                      file.type === 'application/vnd.ms-excel'
      if (!isExcel) {
        this.$message.error('只能上传Excel文件!')
        return false
      }
      return true
    },
    handleImportSuccess(response, file, fileList) {
      if (response.code === '200') {
        this.$message.success(response.msg || '导入成功')
        this.importVisible = false
        this.load(1)
      } else {
        this.$message.error(response.msg || '导入失败')
      }
    },
    handleImportError(err, file, fileList) {
      this.$message.error('导入失败，请检查文件格式是否正确')
    },
    loadDepartment(){
      // 先查出扁平的部门数组
      this.$request.get('/department/selectAll').then(res => {
        this.departmentList = res.data
      })
      // 再查出树形的部门数组
      this.$request.get('/department/selectTree').then(res => {
        this.departmentTree = res.data
      })
    },
    loadCategory(){
      this.$request.get('/category/selectAll').then(res=>{
        this.categoryList=res.data||[]
      })
    },
    loadStaff(){
      this.$request.get('/staff/selectAll').then(res=>{
        this.staffList=res.data||[]
      })
    },
    handleImgSuccess(response, file, fileList) {
      this.form.img = response.data
    },
    // node 就是department对象
    handleNodeClick(node) {
      this.$set(this.form, 'departmentId', node.id)
      this.$refs.selectTree.blur()
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
            url: this.form.id ? '/assets/update' : '/assets/add',
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
        this.$request.delete('/assets/delete/' + id).then(res => {
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
        this.$request.delete('/assets/delete/batch', {data: this.ids}).then(res => {
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
      this.$request.get('/assets/selectPage', {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          name: this.name,
          category:this.category,
          no:this.no,
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
      this.name = null
      this.category = null
      this.no = null
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