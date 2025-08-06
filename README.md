# 固定资产管理系统

![Vue](https://img.shields.io/badge/Vue-2.x-brightgreen?logo=vue.js)
![SpringBoot](https://img.shields.io/badge/SpringBoot-2.x-green?logo=spring)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql)

## 项目简介

本系统是基于前后端分离架构的企业级固定资产管理解决方案，采用Vue2+SpringBoot+MySQL技术栈实现。系统覆盖资产全生命周期管理，包括入库、领用、维修、盘点、调拨等核心功能，支持企业微信/钉钉集成登录，满足企业资产管理需求。

## 技术架构

### 前端架构

| 技术组件| 版本 | 功能说明|
|------------|------|------------------|
| Vue.js| 2.x| SPA应用框架|
| Element UI | 2.x| UI组件库|
| Vue Router | 3.x| 路由管理|
| Axios| 1.x| HTTP请求库|
| ECharts| 5.x| 数据可视化|

### 后端架构

| 技术组件| 版本| 功能说明|
|-----------|---------|------------------|
| SpringBoot| 2.x| Java应用框架|
| MySQL| 8.0| 数据库存储|
| Maven| 3.6+| 依赖管理|

## 目录结构

### 前端目录

```bash
public/# 静态资源
src/
├── assets/# 图片/样式等静态资源
├── components/# 可复用组件(按钮/表单等)
├── views/# 页面组件(资产管理/入库等)
├── router/# 路由配置(index.js)
├── utils/# 工具类(request.js)
├── App.vue# 根组件
└── main.js# 入口文件
package.json# NPM配置文件
vue.config.js# Vue项目配置
```

### 后端目录

```bash
src/
├── main/
│├── java/
││└── com/
││└── example/
││├── controller/# API控制器
││├── service/# 业务逻辑
││├── model/# 数据模型
││└── repository/# 数据访问
│└── resources/
│└── application.properties # 配置文件
files/# 文件存储(附件/图片)
pom.xml# Maven依赖配置
```

## 功能模块

| 功能模块| 接口路径| HTTP方法 | 功能描述|
|--------------|-----------------------|----------|----------------------|
| 资产管理| /assets| GET| 获取资产列表|
|| /assets/{id}| PUT| 更新资产信息|
|| /assets/{id}| DELETE| 删除资产|
| 资产入库| /assets/in| POST| 新增入库资产|
| 资产领用| /assets/out| POST| 提交领用申请|
| 资产维修| /assets/maintenance| POST| 提交维修记录|
| 资产盘点| /assets/stock| GET| 获取盘点数据|
| 资产调拨| /assets/transfer| POST| 提交调拨申请|
| 用户管理| /users| GET| 获取用户列表|
| 部门管理| /departments| GET| 获取部门列表|
| 第三方登录| /oauth2/authorization | GET| 第三方认证跳转|

## 部署指南

### 前端部署

```bash
# 安装依赖
npm install

# 启动开发服务器 (端口:8080)
npm run serve

# 构建生产版本 (输出到dist目录)
npm run build
```

### 后端部署

```bash
# 编译打包
mvn clean package

# 运行应用 (默认端口:8080)
java -jar target/xm-assets.jar

# 指定配置文件运行
java -jar -Dspring.config.location=classpath:/application-prod.yml target/xm-assets.jar
```

## 环境要求

| 组件| 版本要求|
|-----------|---------------|
| Node.js| ≥ 12.x|
| Java| ≥ 1.8|
| Maven| ≥ 3.6|
| MySQL| ≥ 8.0.19|

## 代码示例

### 后端控制器

```java
@RestController
@RequestMapping("/assets")
public class AssetController {

@Autowired
private AssetService assetService;

// 资产入库接口
@PostMapping("/in")
public ResponseEntity<ApiResponse> addAsset(@Valid @RequestBody Asset asset) {
Asset savedAsset = assetService.saveAsset(asset);
return ResponseEntity.ok(
new ApiResponse(true, "资产入库成功", savedAsset)
);
}

// 获取资产列表
@GetMapping
public ResponseEntity<List<Asset>> getAllAssets(
@RequestParam(defaultValue = "0") int page,
@RequestParam(defaultValue = "10") int size
) {
Page<Asset> assetPage = assetService.getAssets(page, size);
return ResponseEntity.ok()
.header("X-Total-Count", String.valueOf(assetPage.getTotalElements()))
.body(assetPage.getContent());
}
}
```

### 前端API调用

```javascript
// utils/request.js
import axios from 'axios';

const service = axios.create({
baseURL: process.env.VUE_APP_BASE_API,
timeout: 5000
});

// 资产入库
export function addAsset(assetData) {
return service.post('/assets/in', assetData);
}

// 获取资产列表
export function fetchAssets(params) {
return service.get('/assets', { params });
}
```

### Vue组件示例

```vue
<template>
<el-table :data="assets" v-loading="loading">
<el-table-column prop="name" label="资产名称"></el-table-column>
<el-table-column prop="serialNumber" label="资产编号"></el-table-column>
<el-table-column prop="status" label="状态"></el-table-column>
<el-table-column label="操作">
<template #default="scope">
<el-button @click="handleEdit(scope.row)">编辑</el-button>
<el-button type="danger" @click="handleDelete(scope.row)">删除</el-button>
</template>
</el-table-column>
</el-table>
</template>

<script>
import { fetchAssets } from '@/utils/request';

export default {
data() {
return {
assets: [],
loading: false
};
},
mounted() {
this.loadAssets();
},
methods: {
async loadAssets() {
this.loading = true;
try {
const response = await fetchAssets({ page: 1, size: 10 });
this.assets = response.data;
} catch (error) {
this.$message.error('获取资产列表失败');
} finally {
this.loading = false;
}
}
}
};
</script>
```

## 注意事项

### 数据库配置
修改 `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/asset_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=your_username
spring.datasource.password=your_password
```

### 第三方登录配置
在集成管理页面配置以下参数：
- 企业微信/钉钉 App ID
- 企业微信/钉钉 App Secret
- 回调地址

### 路由配置
前端路由文件位置：`src/router/index.js`

```javascript
const routes = [
{
path: '/assets',
name: 'AssetManagement',
component: () => import('../views/AssetManagement.vue')
},
{
path: '/assets/in',
name: 'AssetIn',
component: () => import('../views/AssetIn.vue')
}
];
```

### 跨域配置
开发环境配置代理 (`vue.config.js`):

```javascript
module.exports = {
devServer: {
proxy: {
'/api': {
target: 'http://localhost:8080',
changeOrigin: true,
pathRewrite: {
'^/api': ''
}
}
}
}
}
```

## 优化建议

### 安全增强
- 集成Spring Security实现RBAC权限控制
- 添加JWT令牌认证机制
- 实现敏感数据加密存储

### 性能优化
- 引入Redis缓存高频查询数据
- 添加数据库索引优化查询性能
- 实现分页查询和懒加载

### 功能扩展
- 增加资产折旧计算功能
- 实现邮件/消息通知系统
- 添加移动端适配支持
- 开发资产二维码管理功能

### 运维监控
- 集成Spring Boot Admin监控
- 添加Prometheus+Granfana监控
- 实现日志审计功能
