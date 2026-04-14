# 可视化合同模板编辑器 - 完整方案

## 一、项目概述

### 1.1 项目背景
企业在日常运营中需要大量使用各类合同模板，传统的合同模板编辑方式效率低下、难以统一管理、缺少可视化编辑能力。本项目旨在开发一个功能完善的可视化合同模板编辑器，支持 DOCX 模板导入、变量拖拽配置、实时预览、合同生成等功能。

### 1.2 项目目标
- 提供直观的可视化编辑界面
- 支持 DOCX 模板导入和解析
- 实现变量的拖拽式配置
- 支持模板版本管理
- 提供实时预览功能
- 支持合同生成（DOCX/PDF）
- 提供完善的 API 接口

## 二、技术架构

### 2.1 技术栈

#### 前端技术栈
- **框架**: Vue 3 + TypeScript
- **UI 组件库**: Element Plus
- **富文本编辑器**: Quill / TinyMCE
- **拖拽库**: Vue.Draggable / Sortable.js
- **状态管理**: Pinia
- **路由**: Vue Router 4
- **HTTP 客户端**: Axios
- **DOCX 预览**: Docx-preview.js
- **构建工具**: Vite 4

#### 后端技术栈
- **框架**: Spring Boot 2.7.x / 3.x
- **语言**: Java 8+ / 17
- **文档处理**: Apache POI 5.2.x / Docx4j
- **PDF 生成**: iText 7 / OpenPDF / LibreOffice
- **数据库**: MySQL 8.0
- **ORM**: MyBatis Plus
- **文件存储**: MinIO / 阿里云 OSS
- **缓存**: Redis
- **API 文档**: Knife4j / Swagger

### 2.2 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        前端层 (Vue 3)                        │
├─────────────────────────────────────────────────────────────┤
│  模板编辑页面  │  变量管理页面  │  合同生成页面  │  预览页面  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      API 网关层                                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      业务服务层 (Spring Boot)                  │
├─────────────────────────────────────────────────────────────┤
│  模板服务  │  变量服务  │  合同服务  │  预览服务  │  存储服务 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      数据层                                    │
├─────────────────────────────────────────────────────────────┤
│  MySQL (业务数据) │  Redis (缓存) │  MinIO (文件存储)         │
└─────────────────────────────────────────────────────────────┘
```

## 三、功能模块设计

### 3.1 核心功能模块

#### 3.1.1 模板管理模块
- **模板列表**: 分页展示所有模板，支持搜索、筛选
- **模板上传**: 支持 DOCX 文件上传
- **模板解析**: 解析 DOCX 文件，提取文本内容、段落、表格等
- **模板编辑**: 可视化编辑模板内容
- **模板删除**: 逻辑删除模板
- **模板导出**: 导出编辑后的模板
- **模板版本**: 支持模板版本管理、版本回滚

#### 3.1.2 变量管理模块
- **变量定义**: 定义模板中使用的变量
  - 文本变量
  - 日期变量
  - 数字变量
  - 布尔变量
  - 列表变量（单选/多选）
  - 图片变量
  - 表格变量
- **变量拖拽**: 将变量拖拽到模板中
- **变量配置**: 配置变量的默认值、校验规则、数据源
- **变量分组**: 对变量进行分组管理

#### 3.1.3 模板编辑模块
- **富文本编辑**: 支持文本格式化、样式设置
- **表格编辑**: 支持表格创建、行列调整、样式设置
- **图片插入**: 支持图片插入、调整、替换
- **变量拖拽**: 将变量从变量面板拖拽到编辑器中
- **变量高亮**: 在编辑器中高亮显示变量
- **变量编辑**: 点击变量可编辑变量属性
- **撤销/重做**: 支持编辑操作的撤销和重做
- **实时预览**: 编辑时实时预览模板效果

#### 3.1.4 合同生成模块
- **变量填充**: 根据配置的变量值填充模板
- **合同预览**: 预览生成的合同
- **合同下载**: 下载生成的合同（DOCX/PDF）
- **批量生成**: 支持批量生成合同
- **合同归档**: 将生成的合同归档存储

#### 3.1.5 权限管理模块
- **用户管理**: 用户增删改查
- **角色管理**: 角色增删改查
- **权限控制**: 基于角色的权限控制
- **模板权限**: 模板的查看、编辑、删除权限

### 3.2 数据库设计

#### 3.2.1 模板表 (contract_template)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | BIGINT | 主键 |
| name | VARCHAR(200) | 模板名称 |
| description | TEXT | 模板描述 |
| category_id | BIGINT | 分类ID |
| file_path | VARCHAR(500) | 模板文件路径 |
| content | TEXT | 模板内容（JSON） |
| status | TINYINT | 状态：0-禁用，1-启用 |
| create_by | BIGINT | 创建人 |
| create_time | DATETIME | 创建时间 |
| update_by | BIGINT | 更新人 |
| update_time | DATETIME | 更新时间 |
| is_deleted | TINYINT | 是否删除：0-否，1-是 |

#### 3.2.2 变量表 (contract_variable)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | BIGINT | 主键 |
| template_id | BIGINT | 模板ID |
| name | VARCHAR(100) | 变量名称 |
| code | VARCHAR(100) | 变量编码（唯一） |
| type | VARCHAR(50) | 变量类型：text, date, number, boolean, list, image, table |
| default_value | TEXT | 默认值 |
| required | TINYINT | 是否必填：0-否，1-是 |
| validation_rule | TEXT | 校验规则（JSON） |
| data_source | TEXT | 数据源配置（JSON） |
| sort | INT | 排序 |
| create_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |

#### 3.2.3 合同表 (contract)
| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | BIGINT | 主键 |
| template_id | BIGINT | 模板ID |
| name | VARCHAR(200) | 合同名称 |
| contract_no | VARCHAR(100) | 合同编号 |
| variable_values | TEXT | 变量值（JSON） |
| file_path | VARCHAR(500) | 合同文件路径 |
| status | TINYINT | 状态：0-草稿，1-已生成，2-已归档 |
| create_by | BIGINT | 创建人 |
| create_time | DATETIME | 创建时间 |
| update_by | BIGINT | 更新人 |
| update_time | DATETIME | 更新时间 |

## 四、实现步骤

### 第一阶段：项目搭建（1-2周）

#### 1.1 前端项目搭建
1. 使用 Vite 创建 Vue 3 + TypeScript 项目
2. 配置 Element Plus、Pinia、Vue Router、Axios
3. 配置代码规范（ESLint、Prettier）
4. 搭建基础布局（侧边栏、顶部导航、主内容区）
5. 配置代理和环境变量

#### 1.2 后端项目搭建
1. 使用 Spring Initializr 创建 Spring Boot 项目
2. 配置 MyBatis Plus、MySQL、Redis
3. 配置 MinIO 文件存储
4. 搭建基础架构（统一返回格式、异常处理、日志配置）
5. 配置 Knife4j API 文档

#### 1.3 数据库初始化
1. 创建数据库和表结构
2. 初始化基础数据（用户、角色、权限）
3. 配置数据库连接池

### 第二阶段：核心功能开发（3-4周）

#### 2.1 模板管理功能
1. 实现模板列表页面（分页、搜索、筛选）
2. 实现模板上传功能（DOCX 文件上传）
3. 实现模板解析功能（使用 Apache POI 解析 DOCX）
4. 实现模板删除和导出功能
5. 实现后端模板管理 API

#### 2.2 变量管理功能
1. 实现变量定义界面（支持多种变量类型）
2. 实现变量分组功能
3. 实现变量配置界面（默认值、校验规则、数据源）
4. 实现变量拖拽功能（Vue.Draggable）
5. 实现后端变量管理 API

#### 2.3 富文本编辑器集成
1. 集成 Quill / TinyMCE 富文本编辑器
2. 实现表格编辑功能
3. 实现图片插入和管理功能
4. 实现变量高亮显示
5. 实现撤销/重做功能

### 第三阶段：可视化编辑（3-4周）

#### 3.1 DOCX 导入和解析
1. 实现 DOCX 文件解析（Apache POI / Docx4j）
2. 提取段落、表格、图片等元素
3. 将 DOCX 内容转换为编辑器可编辑格式
4. 保留原有样式和格式

#### 3.2 变量拖拽配置
1. 实现变量面板（展示所有可用变量）
2. 实现拖拽功能（将变量拖拽到编辑器）
3. 实现变量占位符渲染（在编辑器中显示变量）
4. 实现变量编辑（点击变量可编辑属性）

#### 3.3 实时预览
1. 实现模板预览功能
2. 实现变量填充预览
3. 实现 DOCX 预览（Docx-preview.js）
4. 实现实时更新（编辑时自动刷新预览）

### 第四阶段：合同生成（2-3周）

#### 4.1 变量填充
1. 实现变量值输入界面
2. 实现变量校验
3. 实现变量值填充逻辑
4. 处理特殊类型变量（列表、图片、表格）

#### 4.2 DOCX 生成
1. 实现 DOCX 文件生成（Apache POI）
2. 保留模板样式
3. 处理图片插入
4. 处理表格填充

#### 4.3 PDF 生成
1. 实现 DOCX 转 PDF（iText 7 / LibreOffice）
2. 配置 PDF 生成参数
3. 优化 PDF 生成性能

#### 4.4 批量生成
1. 实现批量生成界面
2. 实现批量生成逻辑
3. 实现生成进度展示
4. 实现批量下载

### 第五阶段：完善和优化（2-3周）

#### 5.1 权限管理
1. 实现用户管理功能
2. 实现角色管理功能
3. 实现权限控制（基于 RBAC）
4. 实现模板权限

#### 5.2 版本管理
1. 实现模板版本记录
2. 实现版本对比
3. 实现版本回滚
4. 实现版本历史查看

#### 5.3 性能优化
1. 优化 DOCX 解析性能
2. 优化 PDF 生成性能
3. 实现模板缓存
4. 实现懒加载

#### 5.4 测试和文档
1. 编写单元测试
2. 编写接口测试
3. 编写用户操作手册
4. 编写开发文档

## 五、核心技术实现方案

### 5.1 DOCX 解析方案

#### 方案一：Apache POI
- **优点**：功能强大、社区活跃、文档丰富
- **缺点**：API 复杂、学习曲线陡峭、内存占用较高
- **适用场景**：需要深度操作 DOCX 文档

```java
// 示例代码
XWPFDocument document = new XWPFDocument(new FileInputStream("template.docx"));

// 遍历段落
for (XWPFParagraph paragraph : document.getParagraphs()) {
    String text = paragraph.getText();
    // 处理段落内容
}

// 遍历表格
for (XWPFTable table : document.getTables()) {
    for (XWPFTableRow row : table.getRows()) {
        for (XWPFTableCell cell : row.getTableCells()) {
            String text = cell.getText();
            // 处理单元格内容
        }
    }
}
```

#### 方案二：Docx4j
- **优点**：API 更简洁、基于 JAXB、支持 XML 操作
- **缺点**：文档相对较少、社区不如 POI 活跃
- **适用场景**：需要操作 DOCX 的 XML 结构

### 5.2 富文本编辑器方案

#### 方案一：Quill
- **优点**：轻量、模块化、支持 Delta 格式
- **缺点**：表格功能较弱、需要插件扩展
- **适用场景**：轻量级编辑需求

#### 方案二：TinyMCE
- **优点**：功能强大、表格支持好、插件丰富
- **缺点**：体积较大、商业版收费
- **适用场景**：需要强大编辑功能

#### 方案三：CKEditor
- **优点**：功能强大、表格支持好、可定制性强
- **缺点**：配置复杂、体积较大
- **适用场景**：企业级应用

### 5.3 变量拖拽方案

使用 Vue.Draggable + Sortable.js 实现变量拖拽：

```vue
<template>
  <div class="editor-container">
    <!-- 变量面板 -->
    <div class="variable-panel">
      <draggable
        v-model="variables"
        :group="{ name: 'variables', pull: 'clone', put: false }"
        :clone="cloneVariable"
      >
        <div
          v-for="variable in variables"
          :key="variable.id"
          class="variable-item"
          draggable="true"
        >
          {{ variable.name }}
        </div>
      </draggable>
    </div>
    
    <!-- 编辑器 -->
    <div class="editor">
      <div
        class="editor-content"
        @drop="handleDrop"
        @dragover="handleDragOver"
      >
        <!-- 编辑器内容 -->
      </div>
    </div>
  </div>
</template>

<script setup>
import draggable from 'vuedraggable'

const variables = ref([
  { id: 1, name: '甲方名称', code: 'partyAName', type: 'text' },
  { id: 2, name: '乙方名称', code: 'partyBName', type: 'text' },
  { id: 3, name: '合同金额', code: 'contractAmount', type: 'number' }
])

const cloneVariable = (variable) => {
  return { ...variable }
}

const handleDrop = (e) => {
  e.preventDefault()
  const variableData = e.dataTransfer.getData('variable')
  const variable = JSON.parse(variableData)
  // 在编辑器中插入变量占位符
}

const handleDragOver = (e) => {
  e.preventDefault()
}
</script>
```

### 5.4 DOCX 预览方案

使用 Docx-preview.js 实现 DOCX 在线预览：

```vue
<template>
  <div class="docx-preview">
    <div ref="previewContainer"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { renderAsync } from 'docx-preview'

const previewContainer = ref(null)

onMounted(async () => {
  const response = await fetch('/api/templates/1/file')
  const blob = await response.blob()
  await renderAsync(blob, previewContainer.value)
})
</script>
```

### 5.5 PDF 生成方案

#### 方案一：iText 7
- **优点**：功能强大、支持丰富的 PDF 操作
- **缺点**：AGPL 协议（商业使用需要授权）
- **适用场景**：需要深度 PDF 操作

#### 方案二：LibreOffice
- **优点**：开源免费、转换效果好
- **缺点**：需要安装 LibreOffice、性能一般
- **适用场景**：对转换效果要求高

```java
// 使用 LibreOffice 转换 DOCX 到 PDF
String command = "libreoffice --headless --convert-to pdf --outdir /output /input/template.docx";
Process process = Runtime.getRuntime().exec(command);
process.waitFor();
```

#### 方案三：OpenPDF
- **优点**：Apache 协议、轻量
- **缺点**：功能相对简单
- **适用场景**：简单的 PDF 生成需求

## 六、API 接口设计

### 6.1 模板管理接口

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 模板列表 | GET | /api/templates | 获取模板列表 |
| 模板详情 | GET | /api/templates/{id} | 获取模板详情 |
| 上传模板 | POST | /api/templates | 上传模板 |
| 更新模板 | PUT | /api/templates/{id} | 更新模板 |
| 删除模板 | DELETE | /api/templates/{id} | 删除模板 |
| 导出模板 | GET | /api/templates/{id}/export | 导出模板 |
| 模板版本 | GET | /api/templates/{id}/versions | 获取模板版本 |
| 版本回滚 | POST | /api/templates/{id}/rollback | 版本回滚 |

### 6.2 变量管理接口

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 变量列表 | GET | /api/templates/{templateId}/variables | 获取变量列表 |
| 变量详情 | GET | /api/variables/{id} | 获取变量详情 |
| 创建变量 | POST | /api/variables | 创建变量 |
| 更新变量 | PUT | /api/variables/{id} | 更新变量 |
| 删除变量 | DELETE | /api/variables/{id} | 删除变量 |

### 6.3 合同管理接口

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 合同列表 | GET | /api/contracts | 获取合同列表 |
| 合同详情 | GET | /api/contracts/{id} | 获取合同详情 |
| 创建合同 | POST | /api/contracts | 创建合同 |
| 更新合同 | PUT | /api/contracts/{id} | 更新合同 |
| 删除合同 | DELETE | /api/contracts/{id} | 删除合同 |
| 生成合同 | POST | /api/contracts/{id}/generate | 生成合同 |
| 下载合同 | GET | /api/contracts/{id}/download | 下载合同 |
| 批量生成 | POST | /api/contracts/batch-generate | 批量生成合同 |

## 七、部署方案

### 7.1 开发环境

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root123456
      MYSQL_DATABASE: contract_editor
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  minio:
    image: minio/minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio-data:/data
    command: server --console-address ":9001" /data

volumes:
  mysql-data:
  minio-data:
```

### 7.2 生产环境

1. **前端部署**：使用 Nginx 部署前端静态资源
2. **后端部署**：使用 Docker 部署 Spring Boot 应用
3. **数据库**：MySQL 主从复制
4. **缓存**：Redis 集群
5. **文件存储**：MinIO 集群 / 阿里云 OSS
6. **负载均衡**：Nginx / SLB

## 八、风险和注意事项

### 8.1 技术风险
- **DOCX 解析复杂**：DOCX 格式复杂，解析难度大
- **样式兼容性**：不同版本 Word 生成的 DOCX 样式可能有差异
- **PDF 转换效果**：DOCX 转 PDF 可能存在样式不一致
- **性能问题**：大文件解析和 PDF 生成可能耗时较长

### 8.2 应对措施
- **充分测试**：测试不同版本、不同格式的 DOCX 文件
- **降级方案**：提供手动编辑的降级方案
- **异步处理**：大文件处理使用异步方式
- **缓存优化**：缓存解析结果和常用模板

## 九、总结

本方案提供了一个完整的可视化合同模板编辑器的实现方案，涵盖了从项目搭建到上线部署的全过程。通过本方案的实施，可以实现一个功能完善、用户体验良好的合同模板编辑器，支持 DOCX 导入、变量拖拽配置、实时预览、合同生成等核心功能。

项目开发周期预计为 12-16 周，分为五个阶段进行，可以根据实际情况进行调整和优化。
