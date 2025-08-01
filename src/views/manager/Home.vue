<template>
  <div class="dashboard-container">
    <!-- 欢迎卡片 -->
    <div class="welcome-card">
      <div class="welcome-text">您好，{{ user?.name }}！欢迎使用固定资产管理系统</div>
      <div class="current-date">{{ currentDate }}</div>
    </div>

    <!-- 资产概览卡片 -->
    <div class="stats-row">
      <div class="stat-card">
        <div class="stat-title">固定资产总数</div>
        <div class="stat-value">{{ stats.totalAssets || 0 }} 件</div>
        <div class="stat-trend">
          <i class="el-icon-caret-top" style="color:#67C23A"></i>
          <span style="color:#67C23A">+{{ stats.assetGrowthRate || 0 }}%</span> 较上月
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-title">维修中资产</div>
        <div class="stat-value" style="color:#E6A23C">{{ stats.underRepair || 0 }} 件</div>
        <div class="stat-trend">
          <i class="el-icon-caret-bottom" style="color:#F56C6C"></i>
          <span style="color:#F56C6C">-{{ stats.repairDecreaseRate || 0 }}%</span> 较上月
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-title">资产总价值</div>
        <div class="stat-value">{{ formatCurrency(stats.totalValue) }}</div>
        <div class="stat-trend">
          <i class="el-icon-caret-top" style="color:#67C23A"></i>
          <span style="color:#67C23A">+{{ stats.valueGrowthRate || 0 }}%</span> 较上月
        </div>
      </div>
      
      <div class="stat-card">
        <div class="stat-title">盘点次数</div>
        <div class="stat-value">{{ stats.inventoryCount || 0 }} 次</div>
        <div class="stat-trend">
          <i class="el-icon-caret-top" style="color:#67C23A"></i>
          <span style="color:#67C23A">+{{ stats.inventoryGrowthRate || 0 }}%</span> 较上月
        </div>
      </div>
    </div>

    <!-- 主要图表区域 -->
    <div class="charts-row">
      <!-- 资产状态分布 -->
      <div class="chart-card">
        <div class="chart-header">
          <div class="chart-title">资产状态分布</div>
          <el-select v-model="statusPeriod" size="mini" style="width:100px" @change="loadStatusChart">
            <el-option label="本月" value="month"></el-option>
            <el-option label="本季度" value="quarter"></el-option>
            <el-option label="本年" value="year"></el-option>
          </el-select>
        </div>
        <div id="status-chart" style="height:300px"></div>
      </div>
      
      <!-- 报修统计 -->
      <div class="chart-card">
        <div class="chart-header">
          <div class="chart-title">报修统计</div>
          <el-select v-model="repairPeriod" size="mini" style="width:100px" @change="loadRepairChart">
            <el-option label="近7天" value="7d"></el-option>
            <el-option label="近30天" value="30d"></el-option>
            <el-option label="近90天" value="90d"></el-option>
          </el-select>
        </div>
        <div id="repair-chart" style="height:300px"></div>
      </div>
    </div>
    
    <!-- 表格区域 -->
    <div class="tables-row">
      <!-- 固定资产明细 - 修改宽度设置 -->
      <div class="table-card">
        <div class="table-header">
          <div class="table-title">固定资产明细</div>
          <el-button size="mini" type="primary" plain @click="$router.push('/asset')">查看更多</el-button>
        </div>
        <!-- 移除固定宽度，添加滚动容器 -->
        <div class="table-container">
          <el-table :data="assets" height="300" style="min-width: 800px">
            <!-- 移除width属性，添加min-width -->
            <el-table-column prop="no" label="资产编号" min-width="140" align="center" header-align="center"></el-table-column>
            <el-table-column prop="name" label="资产名称" min-width="120" align="center" header-align="center"></el-table-column>
            <el-table-column prop="category" label="类型" min-width="100" align="center" header-align="center"></el-table-column>
            <el-table-column prop="model" label="规格型号" min-width="140" align="center" header-align="center"></el-table-column>
            <el-table-column prop="date" label="购置日期" min-width="100" align="center" header-align="center"></el-table-column>
            <el-table-column prop="money" label="成本单价" min-width="100" align="center" header-align="center">
              <template v-slot="scope">
                {{ formatCurrency(scope.row.money) }}
              </template>
            </el-table-column>
            <el-table-column prop="location" label="存放地点" min-width="120" align="center" header-align="center"></el-table-column>
            <el-table-column prop="status" label="状态" min-width="100" align="center" header-align="center">
              <template v-slot="scope">
                <el-tag size="mini" :type="getStatusTagType(scope.row.status)" align="center">
                  {{ scope.row.status }}
                </el-tag>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
      
      <!-- 报修记录 - 修改宽度设置 -->
      <div class="table-card">
        <div class="table-header">
          <div class="table-title">报修记录</div>
          <el-button size="mini" type="primary" plain @click="$router.push('/repair')">查看更多</el-button>
        </div>
        <!-- 移除固定宽度，添加滚动容器 -->
        <div class="table-container">
          <el-table :data="repairs" height="300" style="min-width: 700px">
            <el-table-column prop="id" label="报修单号" min-width="140" align="center" header-align="center"></el-table-column>
            <el-table-column prop="assetsName" label="资产名称" min-width="120" align="center" header-align="center"></el-table-column>
            <el-table-column prop="date" label="报修日期" min-width="100" align="center" header-align="center"></el-table-column>
            <el-table-column prop="faultDesc" label="故障描述" min-width="150" align="center" header-align="center"></el-table-column>
            <el-table-column prop="staffName" label="报修人" min-width="100" align="center" header-align="center"></el-table-column>
            <el-table-column prop="status" label="状态" min-width="100" align="center" header-align="center">
              <template v-slot="scope">
                <el-tag size="mini" :type="getResultTagType(scope.row.status)" align="center">
                  {{ scope.row.status }}
                </el-tag>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </div>
    </div>
    
    <!-- 底部统计卡片 -->
    <div class="bottom-cards">
      <div class="bottom-card">
        <div class="card-title">盘点统计</div>
        <div style="display:flex; height:200px">
          <div id="inventory-chart" style="flex:1; height:100%"></div>
          <div style="flex:1; padding:15px">
            <div style="margin-bottom:20px">
              <div style="font-size:14px; color:#909399">盘点状态分布</div>
              <div style="display:flex; margin-top:10px">
                <div style="flex:1; text-align:center">
                  <div style="font-size:24px; font-weight:bold; color:#67C23A">{{ stats.normalInventory || 0 }}</div>
                  <div style="font-size:12px; color:#909399">正常</div>
                </div>
                <div style="flex:1; text-align:center">
                  <div style="font-size:24px; font-weight:bold; color:#E6A23C">{{ stats.abnormalInventory || 0 }}</div>
                  <div style="font-size:12px; color:#909399">异常</div>
                </div>
              </div>
            </div>
            <div>
              <div style="font-size:14px; color:#909399">盘点趋势</div>
              <div style="font-size:24px; font-weight:bold; margin-top:10px">{{ stats.inventoryCount || 0 }} 次</div>
              <div style="font-size:12px; color:#909399">本月盘点次数</div>
            </div>
          </div>
        </div>
      </div>
      
      <div class="bottom-card">
        <div class="card-title">公告列表</div>
        <el-timeline style="padding:15px">
          <el-timeline-item 
            v-for="item in notices" 
            :key="item.id" 
            :timestamp="item.time"
            placement="top"
          >
            <el-card shadow="hover" style="cursor:pointer" @click="showNotice(item)">
              <div style="font-weight:bold">{{ item.title }}</div>
              <div style="margin-top:5px; color:#606266; font-size:13px">
                {{ item.content.substring(0, 50) }}...
              </div>
            </el-card>
          </el-timeline-item>
        </el-timeline>
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'

export default {
  name: 'Home',
  data() {
    return {
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      currentDate: new Date().toLocaleDateString('zh-CN', { 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric',
        weekday: 'long'
      }),
      statusPeriod: 'month',
      repairPeriod: '30d',
      stats: {
        totalAssets: 0,
        underRepair: 0,
        totalValue: 0,
        inventoryCount: 0,
        normalInventory: 0,
        abnormalInventory: 0,
        assetGrowthRate: 0,
        repairDecreaseRate: 0,
        valueGrowthRate: 0,
        inventoryGrowthRate: 0
      },
      assets: [],
      repairs: [],
      notices: [],
      statusChart: null,
      repairChart: null,
      inventoryChart: null
    }
  },
  created() {
    this.loadData();
    this.loadNotices();
  },
  mounted() {
    this.initCharts();
  },
  methods: {
    formatCurrency(value) {
      return '¥' + (value || 0).toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    },
    
    getStatusTagType(status) {
      switch(status) {
        case '正常': return 'success';
        case '维修中': return 'warning';
        case '报废': return 'danger';
        case '闲置': return 'info';
        default: return '';
      }
    },
    
    getResultTagType(result) {
      switch(result) {
        case '正常': 
        case '已完成': 
          return 'success';
        case '处理中': 
        case '待审核': 
          return 'warning';
        case '已关闭': 
          return '';
        default: 
          return 'info';
      }
    },
    
    initCharts() {
      this.statusChart = echarts.init(document.getElementById('status-chart'));
      this.repairChart = echarts.init(document.getElementById('repair-chart'));
      this.inventoryChart = echarts.init(document.getElementById('inventory-chart'));
      
      window.addEventListener('resize', () => {
        this.statusChart.resize();
        this.repairChart.resize();
        this.inventoryChart.resize();
      });
      
      this.loadStatusChart();
      this.loadRepairChart();
      this.loadInventoryChart();
    },
    
    async loadData() {
      try {
        // 加载统计概览数据
        const statsRes = await this.$request.get('/dashboard/stats');
        if (statsRes.code === '200') {
          this.stats = statsRes.data;
        }
        
        // 加载固定资产明细
        const assetsRes = await this.$request.get('/assets/selectAll', {
          params: { pageSize: 10 }
        });
        if (assetsRes.code === '200') {
          this.assets = assetsRes.data || [];
        }
        
        // 加载报修记录
        const repairsRes = await this.$request.get('/assetsRepair/recent');
        if (repairsRes.code === '200') {
          this.repairs = repairsRes.data;
        }
      } catch (error) {
        console.error('加载数据失败:', error);
        this.$message.error('数据加载失败');
      }
    },
    
    async loadNotices() {
      try {
        const res = await this.$request.get('/notice/selectAll');
        if (res.code === '200') {
          this.notices = res.data || [];
        }
      } catch (error) {
        console.error('加载公告失败:', error);
      }
    },
    
    async loadStatusChart() {
      try {
        const res = await this.$request.get('/dashboard/status-distribution', {
          params: { period: this.statusPeriod }
        });
        
        if (res.code === '200') {
          const option = {
            tooltip: {
              trigger: 'item',
              formatter: '{a} <br/>{b}: {c} ({d}%)'
            },
            legend: {
              orient: 'vertical',
              right: 10,
              top: 'center',
              data: res.data.map(item => item.name)
            },
            series: [
              {
                name: '资产状态',
                type: 'pie',
                radius: ['40%', '70%'],
                center: ['40%', '50%'],
                avoidLabelOverlap: false,
                itemStyle: {
                  borderRadius: 10,
                  borderColor: '#fff',
                  borderWidth: 2
                },
                label: {
                  show: true,
                  formatter: '{b}: {c} ({d}%)'
                },
                emphasis: {
                  label: {
                    show: true,
                    fontSize: '16',
                    fontWeight: 'bold'
                  }
                },
                labelLine: {
                  show: true
                },
                data: res.data
              }
            ]
          };
          
          this.statusChart.setOption(option);
        }
      } catch (error) {
        console.error('加载状态分布失败:', error);
      }
    },
    
    async loadRepairChart() {
      try {
        const res = await this.$request.get('/dashboard/repair-stats', {
          params: { period: this.repairPeriod }
        });
        
        if (res.code === '200') {
          const option = {
            tooltip: {
              trigger: 'axis'
            },
            legend: {
              data: ['报修数量', '完成数量']
            },
            grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
            },
            xAxis: {
              type: 'category',
              boundaryGap: false,
              data: res.data.labels
            },
            yAxis: {
              type: 'value'
            },
            series: [
              {
                name: '报修数量',
                type: 'line',
                smooth: true,
                symbol: 'circle',
                symbolSize: 8,
                lineStyle: {
                  width: 3
                },
                itemStyle: {
                  color: '#409EFF'
                },
                data: res.data.repairCounts
              },
              {
                name: '完成数量',
                type: 'line',
                smooth: true,
                symbol: 'circle',
                symbolSize: 8,
                lineStyle: {
                  width: 3
                },
                itemStyle: {
                  color: '#67C23A'
                },
                data: res.data.completedCounts
              }
            ]
          };
          
          this.repairChart.setOption(option);
        }
      } catch (error) {
        console.error('加载报修统计失败:', error);
      }
    },
    
    async loadInventoryChart() {
      try {
        const res = await this.$request.get('/dashboard/inventory-trend');
        
        if (res.code === '200') {
          const option = {
            tooltip: {
              trigger: 'axis'
            },
            grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
            },
            xAxis: {
              type: 'category',
              data: res.data.labels
            },
            yAxis: {
              type: 'value'
            },
            series: [
              {
                name: '盘点次数',
                type: 'bar',
                barWidth: '60%',
                itemStyle: {
                  color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                    { offset: 0, color: '#83bff6' },
                    { offset: 0.5, color: '#188df0' },
                    { offset: 1, color: '#188df0' }
                  ])
                },
                data: res.data.values
              }
            ]
          };
          
          this.inventoryChart.setOption(option);
        }
      } catch (error) {
        console.error('加载盘点统计失败:', error);
      }
    },
    
    showNotice(notice) {
      this.$alert(notice.content, notice.title, {
        confirmButtonText: '确定',
        customClass: 'notice-detail'
      });
    }
  }
}
</script>

<style scoped>
.dashboard-container {
  padding: 20px;
}

.welcome-card {
  background: linear-gradient(135deg, #409EFF, #64b5f6);
  color: white;
  padding: 20px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.welcome-text {
  font-size: 20px;
  font-weight: bold;
}

.current-date {
  font-size: 16px;
  opacity: 0.9;
}

.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  border-left: 4px solid #409EFF;
}

.stat-card:nth-child(2) {
  border-left-color: #E6A23C;
}

.stat-card:nth-child(3) {
  border-left-color: #67C23A;
}

.stat-card:nth-child(4) {
  border-left-color: #909399;
}

.stat-title {
  font-size: 14px;
  color: #606266;
  margin-bottom: 10px;
}

.stat-value {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 5px;
}

.stat-trend {
  font-size: 12px;
  color: #909399;
}

.charts-row {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

.chart-card {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.chart-title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.tables-row {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

/* 添加表格容器样式 */
.table-container {
  overflow-x: auto;
}

/* 添加响应式处理 */
@media (max-width: 1200px) {
  .tables-row {
    grid-template-columns: 1fr !important;
  }
  
  .charts-row {
    grid-template-columns: 1fr !important;
  }
  
  .bottom-cards {
    grid-template-columns: 1fr !important;
  }
  
  .stat-card {
    min-width: calc(50% - 10px);
  }
}

@media (max-width: 768px) {
  .stats-row {
    grid-template-columns: repeat(2, 1fr) !important;
  }
}

@media (max-width: 480px) {
  .stats-row {
    grid-template-columns: 1fr !important;
  }
  
  .table-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .table-header .el-button {
    margin-top: 10px;
    align-self: flex-end;
  }
}

.table-card {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.table-title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
}

.bottom-cards {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.bottom-card {
  background: white;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.card-title {
  font-size: 16px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 15px;
}

/* 表格单元格垂直居中 */
.el-table >>> .cell {
  display: flex;
  align-items: center;
  height: 100%;
  justify-content: center;
}

/* 表头垂直居中 */
.el-table >>> th > .cell {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>

<style>
.notice-detail {
  width: 60% !important;
}
.notice-detail .el-message-box__content {
  white-space: pre-wrap;
  line-height: 1.6;
  max-height: 60vh;
  overflow-y: auto;
}
</style>