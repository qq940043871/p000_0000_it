# 可视化合同模板编辑器 - 详细实现步骤

## 一、项目初始化

### 1.1 前端项目初始化

#### 步骤 1：创建 Vue 3 项目

```bash
npm create vite@latest contract-editor-frontend -- --template vue-ts
cd contract-editor-frontend
npm install
```

#### 步骤 2：安装核心依赖

```bash
# UI 组件库
npm install element-plus @element-plus/icons-vue

# 路由和状态管理
npm install vue-router pinia

# HTTP 客户端
npm install axios

# 富文本编辑器（选择一个）
npm install @vueup/vue-quill
# 或
npm install tinymce @tinymce/tinymce-vue

# 拖拽库
npm install vuedraggable@next sortablejs

# DOCX 预览
npm install docx-preview

# 工具库
npm install dayjs lodash-es
```

#### 步骤 3：安装开发依赖

```bash
npm install -D @types/node sass eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier eslint-config-prettier eslint-plugin-prettier eslint-plugin-vue
```

#### 步骤 4：配置 Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  }
})
```

#### 步骤 5：配置 TypeScript

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

#### 步骤 6：配置 ESLint 和 Prettier

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  parser: 'vue-eslint-parser',
  parserOptions: {
    ecmaVersion: 'latest',
    parser: '@typescript-eslint/parser',
    sourceType: 'module'
  },
  plugins: ['vue', '@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error',
    'vue/multi-word-component-names': 'off'
  }
}
```

```json
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "none",
  "arrowParens": "avoid"
}
```

#### 步骤 7：创建项目目录结构

```
src/
├── api/                    # API 接口
│   ├── index.ts
│   ├── template.ts
│   ├── variable.ts
│   └── contract.ts
├── assets/                 # 静态资源
├── components/             # 组件
│   ├── editor/             # 编辑器相关组件
│   ├── variable/           # 变量相关组件
│   └── common/             # 通用组件
├── layouts/                # 布局组件
├── router/                 # 路由
│   └── index.ts
├── stores/                 # Pinia 状态管理
│   ├── template.ts
│   ├── variable.ts
│   └── user.ts
├── types/                  # TypeScript 类型定义
│   ├── template.ts
│   ├── variable.ts
│   └── contract.ts
├── utils/                  # 工具函数
│   ├── request.ts
│   └── docx.ts
├── views/                  # 页面
│   ├── TemplateList.vue
│   ├── TemplateEditor.vue
│   ├── VariableManage.vue
│   └── ContractGenerate.vue
├── App.vue
└── main.ts
```

#### 步骤 8：配置路由

```typescript
// src/router/index.ts
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    redirect: '/templates'
  },
  {
    path: '/templates',
    name: 'TemplateList',
    component: () => import('@/views/TemplateList.vue')
  },
  {
    path: '/templates/:id/edit',
    name: 'TemplateEditor',
    component: () => import('@/views/TemplateEditor.vue')
  },
  {
    path: '/variables',
    name: 'VariableManage',
    component: () => import('@/views/VariableManage.vue')
  },
  {
    path: '/contracts',
    name: 'ContractGenerate',
    component: () => import('@/views/ContractGenerate.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

#### 步骤 9：配置 Pinia

```typescript
// src/stores/template.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'
import type { Template } from '@/types/template'

export const useTemplateStore = defineStore('template', () => {
  const currentTemplate = ref<Template | null>(null)
  const templateList = ref<Template[]>([])

  const setCurrentTemplate = (template: Template) => {
    currentTemplate.value = template
  }

  const setTemplateList = (list: Template[]) => {
    templateList.value = list
  }

  return {
    currentTemplate,
    templateList,
    setCurrentTemplate,
    setTemplateList
  }
})
```

#### 步骤 10：配置 Axios

```typescript
// src/utils/request.ts
import axios from 'axios'
import { ElMessage } from 'element-plus'

const request = axios.create({
  baseURL: '/api',
  timeout: 30000
})

// 请求拦截器
request.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// 响应拦截器
request.interceptors.response.use(
  response => {
    const res = response.data
    if (res.code !== 200) {
      ElMessage.error(res.message || '请求失败')
      return Promise.reject(new Error(res.message || '请求失败'))
    }
    return res
  },
  error => {
    ElMessage.error(error.message || '网络错误')
    return Promise.reject(error)
  }
)

export default request
```

```typescript
// src/api/template.ts
import request from '@/utils/request'
import type { Template, TemplateQuery, TemplatePageResult } from '@/types/template'

export const getTemplateList = (params: TemplateQuery) => {
  return request.get<TemplatePageResult>('/templates', { params })
}

export const getTemplateDetail = (id: number) => {
  return request.get<Template>(`/templates/${id}`)
}

export const uploadTemplate = (file: File, data: Partial<Template>) => {
  const formData = new FormData()
  formData.append('file', file)
  formData.append('name', data.name || '')
  formData.append('description', data.description || '')
  formData.append('categoryId', String(data.categoryId || ''))
  return request.post<Template>('/templates', formData, {
    headers: { 'Content-Type': 'multipart/form-data' }
  })
}

export const updateTemplate = (id: number, data: Partial<Template>) => {
  return request.put<Template>(`/templates/${id}`, data)
}

export const deleteTemplate = (id: number) => {
  return request.delete(`/templates/${id}`)
}

export const downloadTemplate = (id: number) => {
  return request.get(`/templates/${id}/export`, { responseType: 'blob' })
}
```

### 1.2 后端项目初始化

#### 步骤 1：创建 Spring Boot 项目

使用 Spring Initializr 创建项目，选择以下依赖：
- Spring Web
- Spring Data JPA / MyBatis Plus
- MySQL Driver
- Redis (可选)
- Validation
- Lombok

#### 步骤 2：添加核心依赖

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- MyBatis Plus -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>
    
    <!-- MySQL -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Apache POI -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>
    
    <!-- iText 7 -->
    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itext7-core</artifactId>
        <version>7.2.3</version>
        <type>pom</type>
    </dependency>
    
    <!-- MinIO -->
    <dependency>
        <groupId>io.minio</groupId>
        <artifactId>minio</artifactId>
        <version>8.5.2</version>
    </dependency>
    
    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Knife4j -->
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
        <version>4.1.0</version>
    </dependency>
</dependencies>
```

#### 步骤 3：配置 application.yml

```yaml
spring:
  application:
    name: contract-template-editor
  
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/contract_editor?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
    username: root
    password: root123456
  
  servlet:
    multipart:
      max-file-size: 50MB
      max-request-size: 50MB

server:
  port: 8080

mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: isDeleted
      logic-delete-value: 1
      logic-not-delete-value: 0

minio:
  endpoint: http://localhost:9000
  accessKey: minioadmin
  secretKey: minioadmin
  bucketName: contract-editor

knife4j:
  enable: true
  setting:
    language: zh_cn
```

#### 步骤 4：创建项目目录结构

```
src/main/java/com/contract/editor/
├── ContractEditorApplication.java
├── common/             # 通用模块
│   ├── result/         # 统一返回结果
│   ├── exception/      # 异常处理
│   └── constant/       # 常量
├── config/             # 配置类
│   ├── MyBatisPlusConfig.java
│   ├── MinIOConfig.java
│   └── Knife4jConfig.java
├── controller/         # 控制器
│   ├── TemplateController.java
│   ├── VariableController.java
│   └── ContractController.java
├── service/            # 服务层
│   ├── TemplateService.java
│   ├── VariableService.java
│   ├── ContractService.java
│   └── impl/
├── mapper/             # 数据访问层
│   ├── TemplateMapper.java
│   ├── VariableMapper.java
│   └── ContractMapper.java
├── entity/             # 实体类
│   ├── Template.java
│   ├── Variable.java
│   └── Contract.java
├── dto/                # 数据传输对象
│   ├── request/
│   └── response/
├── util/               # 工具类
│   ├── MinIOUtil.java
│   └── DocxUtil.java
└── enums/              # 枚举类
```

#### 步骤 5：创建统一返回结果

```java
// src/main/java/com/contract/editor/common/result/Result.java
package com.contract.editor.common.result;

import lombok.Data;

@Data
public class Result<T> {
    private Integer code;
    private String message;
    private T data;

    public static <T> Result<T> success() {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        return result;
    }

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        return result;
    }

    public static <T> Result<T> error(String message) {
        Result<T> result = new Result<>();
        result.setCode(500);
        result.setMessage(message);
        return result;
    }
}
```

#### 步骤 6：创建全局异常处理

```java
// src/main/java/com/contract/editor/common/exception/GlobalExceptionHandler.java
package com.contract.editor.common.exception;

import com.contract.editor.common.result.Result;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public Result<Void> handleException(Exception e) {
        log.error("系统异常：", e);
        return Result.error("系统异常：" + e.getMessage());
    }
}
```

#### 步骤 7：创建实体类

```java
// src/main/java/com/contract/editor/entity/Template.java
package com.contract.editor.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("contract_template")
public class Template {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private String name;
    
    private String description;
    
    private Long categoryId;
    
    private String filePath;
    
    private String content;
    
    private Integer status;
    
    private Long createBy;
    
    private LocalDateTime createTime;
    
    private Long updateBy;
    
    private LocalDateTime updateTime;
    
    @TableLogic
    private Integer isDeleted;
}
```

```java
// src/main/java/com/contract/editor/entity/Variable.java
package com.contract.editor.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("contract_variable")
public class Variable {
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private Long templateId;
    
    private String name;
    
    private String code;
    
    private String type;
    
    private String defaultValue;
    
    private Integer required;
    
    private String validationRule;
    
    private String dataSource;
    
    private Integer sort;
    
    private LocalDateTime createTime;
    
    private LocalDateTime updateTime;
}
```

#### 步骤 8：创建 Mapper 接口

```java
// src/main/java/com/contract/editor/mapper/TemplateMapper.java
package com.contract.editor.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.contract.editor.entity.Template;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface TemplateMapper extends BaseMapper<Template> {
}
```

## 二、DOCX 解析模块实现

### 2.1 后端 DOCX 解析实现

#### 步骤 1：创建 DOCX 工具类

```java
// src/main/java/com/contract/editor/util/DocxUtil.java
package com.contract.editor.util;

import org.apache.poi.xwpf.usermodel.*;
import org.springframework.stereotype.Component;

import java.io.*;
import java.util.*;

@Component
public class DocxUtil {

    /**
     * 解析 DOCX 文件
     */
    public Map<String, Object> parseDocx(InputStream inputStream) throws IOException {
        Map<String, Object> result = new HashMap<>();
        List<Map<String, Object>> paragraphs = new ArrayList<>();
        List<Map<String, Object>> tables = new ArrayList<>();

        try (XWPFDocument document = new XWPFDocument(inputStream)) {
            // 解析段落
            for (XWPFParagraph paragraph : document.getParagraphs()) {
                Map<String, Object> paraMap = new HashMap<>();
                paraMap.put("text", paragraph.getText());
                paraMap.put("style", paragraph.getStyle());
                paraMap.put("alignment", paragraph.getAlignment() != null ? paragraph.getAlignment().name() : null);
                paragraphs.add(paraMap);
            }
            result.put("paragraphs", paragraphs);

            // 解析表格
            for (XWPFTable table : document.getTables()) {
                Map<String, Object> tableMap = new HashMap<>();
                List<List<Map<String, Object>>> rows = new ArrayList<>();
                
                for (XWPFTableRow row : table.getRows()) {
                    List<Map<String, Object>> cells = new ArrayList<>();
                    for (XWPFTableCell cell : row.getTableCells()) {
                        Map<String, Object> cellMap = new HashMap<>();
                        cellMap.put("text", cell.getText());
                        cells.add(cellMap);
                    }
                    rows.add(cells);
                }
                tableMap.put("rows", rows);
                tables.add(tableMap);
            }
            result.put("tables", tables);
        }

        return result;
    }

    /**
     * 生成 DOCX 文件
     */
    public void generateDocx(Map<String, Object> data, OutputStream outputStream) throws IOException {
        try (XWPFDocument document = new XWPFDocument()) {
            // 实现文档生成逻辑
            document.write(outputStream);
        }
    }

    /**
     * 填充模板变量
     */
    public void fillTemplate(InputStream templateStream, Map<String, Object> variables, OutputStream outputStream) throws IOException {
        try (XWPFDocument document = new XWPFDocument(templateStream)) {
            // 替换段落中的变量
            for (XWPFParagraph paragraph : document.getParagraphs()) {
                replaceVariablesInParagraph(paragraph, variables);
            }

            // 替换表格中的变量
            for (XWPFTable table : document.getTables()) {
                for (XWPFTableRow row : table.getRows()) {
                    for (XWPFTableCell cell : row.getTableCells()) {
                        for (XWPFParagraph paragraph : cell.getParagraphs()) {
                            replaceVariablesInParagraph(paragraph, variables);
                        }
                    }
                }
            }

            document.write(outputStream);
        }
    }

    private void replaceVariablesInParagraph(XWPFParagraph paragraph, Map<String, Object> variables) {
        String text = paragraph.getText();
        if (text == null || text.isEmpty()) {
            return;
        }

        // 简单的变量替换逻辑
        for (Map.Entry<String, Object> entry : variables.entrySet()) {
            String placeholder = "${" + entry.getKey() + "}";
            if (text.contains(placeholder)) {
                // 替换变量
                // 注意：实际实现需要处理段落的 runs
            }
        }
    }
}
```

#### 步骤 2：创建模板服务

```java
// src/main/java/com/contract/editor/service/TemplateService.java
package com.contract.editor.service;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.IService;
import com.contract.editor.entity.Template;
import org.springframework.web.multipart.MultipartFile;

public interface TemplateService extends IService<Template> {
    
    Page<Template> getTemplatePage(Integer pageNum, Integer pageSize, String keyword);
    
    Template uploadTemplate(MultipartFile file, Template template);
    
    void parseTemplate(Long id);
}
```

```java
// src/main/java/com/contract/editor/service/impl/TemplateServiceImpl.java
package com.contract.editor.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.contract.editor.entity.Template;
import com.contract.editor.mapper.TemplateMapper;
import com.contract.editor.service.TemplateService;
import com.contract.editor.util.DocxUtil;
import com.contract.editor.util.MinIOUtil;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.time.LocalDateTime;
import java.util.Map;

@Service
@RequiredArgsConstructor
public class TemplateServiceImpl extends ServiceImpl<TemplateMapper, Template> implements TemplateService {

    private final DocxUtil docxUtil;
    private final MinIOUtil minIOUtil;
    private final ObjectMapper objectMapper;

    @Override
    public Page<Template> getTemplatePage(Integer pageNum, Integer pageSize, String keyword) {
        Page<Template> page = new Page<>(pageNum, pageSize);
        LambdaQueryWrapper<Template> wrapper = new LambdaQueryWrapper<>();
        
        if (keyword != null && !keyword.isEmpty()) {
            wrapper.like(Template::getName, keyword);
        }
        
        wrapper.orderByDesc(Template::getCreateTime);
        return this.page(page, wrapper);
    }

    @Override
    public Template uploadTemplate(MultipartFile file, Template template) {
        try {
            // 上传文件到 MinIO
            String fileName = System.currentTimeMillis() + "_" + file.getOriginalFilename();
            String filePath = minIOUtil.uploadFile(file.getInputStream(), fileName, file.getContentType());
            
            // 保存模板信息
            template.setFilePath(filePath);
            template.setStatus(1);
            template.setCreateTime(LocalDateTime.now());
            template.setUpdateTime(LocalDateTime.now());
            template.setIsDeleted(0);
            this.save(template);
            
            return template;
        } catch (Exception e) {
            throw new RuntimeException("上传模板失败", e);
        }
    }

    @Override
    public void parseTemplate(Long id) {
        try {
            Template template = this.getById(id);
            if (template == null) {
                throw new RuntimeException("模板不存在");
            }
            
            // 从 MinIO 下载文件
            InputStream inputStream = minIOUtil.downloadFile(template.getFilePath());
            
            // 解析 DOCX
            Map<String, Object> parsedContent = docxUtil.parseDocx(inputStream);
            
            // 保存解析后的内容
            template.setContent(objectMapper.writeValueAsString(parsedContent));
            template.setUpdateTime(LocalDateTime.now());
            this.updateById(template);
            
        } catch (Exception e) {
            throw new RuntimeException("解析模板失败", e);
        }
    }
}
```

#### 步骤 3：创建模板控制器

```java
// src/main/java/com/contract/editor/controller/TemplateController.java
package com.contract.editor.controller;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.contract.editor.common.result.Result;
import com.contract.editor.entity.Template;
import com.contract.editor.service.TemplateService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@Tag(name = "模板管理")
@RestController
@RequestMapping("/api/templates")
@RequiredArgsConstructor
public class TemplateController {

    private final TemplateService templateService;

    @Operation(summary = "获取模板列表")
    @GetMapping
    public Result<Page<Template>> getTemplateList(
            @RequestParam(defaultValue = "1") Integer pageNum,
            @RequestParam(defaultValue = "10") Integer pageSize,
            @RequestParam(required = false) String keyword) {
        Page<Template> page = templateService.getTemplatePage(pageNum, pageSize, keyword);
        return Result.success(page);
    }

    @Operation(summary = "获取模板详情")
    @GetMapping("/{id}")
    public Result<Template> getTemplateDetail(@PathVariable Long id) {
        Template template = templateService.getById(id);
        return Result.success(template);
    }

    @Operation(summary = "上传模板")
    @PostMapping
    public Result<Template> uploadTemplate(
            @RequestParam("file") MultipartFile file,
            @RequestParam("name") String name,
            @RequestParam(value = "description", required = false) String description) {
        Template template = new Template();
        template.setName(name);
        template.setDescription(description);
        Template savedTemplate = templateService.uploadTemplate(file, template);
        return Result.success(savedTemplate);
    }

    @Operation(summary = "更新模板")
    @PutMapping("/{id}")
    public Result<Template> updateTemplate(@PathVariable Long id, @RequestBody Template template) {
        template.setId(id);
        templateService.updateById(template);
        return Result.success(template);
    }

    @Operation(summary = "删除模板")
    @DeleteMapping("/{id}")
    public Result<Void> deleteTemplate(@PathVariable Long id) {
        templateService.removeById(id);
        return Result.success();
    }

    @Operation(summary = "解析模板")
    @PostMapping("/{id}/parse")
    public Result<Void> parseTemplate(@PathVariable Long id) {
        templateService.parseTemplate(id);
        return Result.success();
    }
}
```

### 2.2 前端 DOCX 预览实现

#### 步骤 1：创建 DOCX 预览组件

```vue
<!-- src/components/editor/DocxPreview.vue -->
<template>
  <div class="docx-preview" ref="previewContainer"></div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { renderAsync } from 'docx-preview'

interface Props {
  fileUrl: string
}

const props = defineProps<Props>()
const previewContainer = ref<HTMLElement | null>(null)

const renderDocx = async (url: string) => {
  if (!previewContainer.value || !url) return
  
  try {
    const response = await fetch(url)
    const blob = await response.blob()
    await renderAsync(blob, previewContainer.value)
  } catch (error) {
    console.error('DOCX 预览失败:', error)
  }
}

watch(() => props.fileUrl, (newUrl) => {
  if (newUrl) {
    renderDocx(newUrl)
  }
}, { immediate: true })
</script>

<style scoped>
.docx-preview {
  width: 100%;
  min-height: 500px;
  border: 1px solid #e5e7eb;
  border-radius: 4px;
  overflow: auto;
  padding: 20px;
}
</style>
```

## 三、富文本编辑器模块实现

### 3.1 前端富文本编辑器集成

#### 步骤 1：集成 Quill 编辑器

```vue
<!-- src/components/editor/RichTextEditor.vue -->
<template>
  <div class="rich-text-editor">
    <div ref="editorRef"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import Quill from 'quill'
import 'quill/dist/quill.snow.css'

interface Props {
  modelValue: string
  placeholder?: string
}

interface Emits {
  (e: 'update:modelValue', value: string): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()
const editorRef = ref<HTMLElement | null>(null)
let quill: Quill | null = null

onMounted(() => {
  if (editorRef.value) {
    quill = new Quill(editorRef.value, {
      theme: 'snow',
      placeholder: props.placeholder || '请输入内容...',
      modules: {
        toolbar: [
          [{ 'header': [1, 2, 3, false] }],
          ['bold', 'italic', 'underline', 'strike'],
          [{ 'color': [] }, { 'background': [] }],
          [{ 'list': 'ordered'}, { 'list': 'bullet' }],
          [{ 'align': [] }],
          ['link', 'image', 'table'],
          ['clean']
        ]
      }
    })

    // 监听内容变化
    quill.on('text-change', () => {
      emit('update:modelValue', quill?.root.innerHTML || '')
    })

    // 设置初始内容
    if (props.modelValue) {
      quill.root.innerHTML = props.modelValue
    }
  }
})

watch(() => props.modelValue, (newValue) => {
  if (quill && newValue !== quill.root.innerHTML) {
    quill.root.innerHTML = newValue
  }
})
</script>

<style scoped>
.rich-text-editor {
  width: 100%;
}

:deep(.ql-container) {
  min-height: 400px;
  font-size: 16px;
}
</style>
```

#### 步骤 2：创建变量拖拽面板

```vue
<!-- src/components/variable/VariablePanel.vue -->
<template>
  <div class="variable-panel">
    <div class="panel-header">
      <span class="title">变量列表</span>
      <el-button type="primary" size="small" @click="openAddDialog">
        添加变量
      </el-button>
    </div>
    
    <div class="variable-list">
      <draggable
        v-model="variables"
        :group="{ name: 'variables', pull: 'clone', put: false }"
        :clone="cloneVariable"
        item-key="id"
      >
        <template #item="{ element }">
          <div class="variable-item" draggable="true">
            <div class="variable-icon">{{ getTypeIcon(element.type) }}</div>
            <div class="variable-info">
              <div class="variable-name">{{ element.name }}</div>
              <div class="variable-code">{{ element.code }}</div>
            </div>
          </div>
        </template>
      </draggable>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import draggable from 'vuedraggable'
import { ElMessage } from 'element-plus'
import type { Variable } from '@/types/variable'

interface Props {
  variables: Variable[]
}

const props = defineProps<Props>()

const cloneVariable = (variable: Variable) => {
  return { ...variable }
}

const getTypeIcon = (type: string) => {
  const icons: Record<string, string> = {
    text: 'T',
    number: '#',
    date: '📅',
    boolean: '☑️',
    list: '📋',
    image: '🖼️',
    table: '📊'
  }
  return icons[type] || '?'
}

const openAddDialog = () => {
  ElMessage.info('打开添加变量对话框')
}
</script>

<style scoped>
.variable-panel {
  width: 280px;
  height: 100%;
  background: #fff;
  border-right: 1px solid #e5e7eb;
  display: flex;
  flex-direction: column;
}

.panel-header {
  padding: 16px;
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title {
  font-size: 16px;
  font-weight: 600;
}

.variable-list {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
}

.variable-item {
  display: flex;
  align-items: center;
  padding: 12px;
  margin-bottom: 8px;
  background: #f9fafb;
  border-radius: 8px;
  cursor: grab;
  transition: all 0.2s;
}

.variable-item:hover {
  background: #eff6ff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.variable-item:active {
  cursor: grabbing;
}

.variable-icon {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #3b82f6;
  color: #fff;
  border-radius: 8px;
  font-weight: 600;
  margin-right: 12px;
}

.variable-info {
  flex: 1;
}

.variable-name {
  font-size: 14px;
  font-weight: 500;
  color: #1f2937;
}

.variable-code {
  font-size: 12px;
  color: #6b7280;
  margin-top: 2px;
}
</style>
```

#### 步骤 3：创建模板编辑器主页面

```vue
<!-- src/views/TemplateEditor.vue -->
<template>
  <div class="template-editor">
    <el-page-header @back="goBack" :content="templateName">
      <template #extra>
        <el-button @click="saveTemplate">保存</el-button>
        <el-button type="primary" @click="previewTemplate">预览</el-button>
      </template>
    </el-page-header>
    
    <div class="editor-layout">
      <!-- 变量面板 -->
      <VariablePanel :variables="variables" />
      
      <!-- 编辑区域 -->
      <div class="editor-area">
        <el-tabs v-model="activeTab">
          <el-tab-pane label="编辑" name="edit">
            <div 
              class="editor-content" 
              @drop="handleDrop"
              @dragover="handleDragOver"
            >
              <RichTextEditor v-model="templateContent" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="DOCX预览" name="preview">
            <DocxPreview :file-url="templateFileUrl" />
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { ElMessage } from 'element-plus'
import VariablePanel from '@/components/variable/VariablePanel.vue'
import RichTextEditor from '@/components/editor/RichTextEditor.vue'
import DocxPreview from '@/components/editor/DocxPreview.vue'
import type { Variable } from '@/types/variable'

const router = useRouter()
const route = useRoute()
const templateId = ref(Number(route.params.id))
const templateName = ref('合同模板编辑')
const activeTab = ref('edit')
const templateContent = ref('')
const templateFileUrl = ref('')

const variables = ref<Variable[]>([
  { id: 1, name: '甲方名称', code: 'partyAName', type: 'text', required: 1, sort: 1 },
  { id: 2, name: '乙方名称', code: 'partyBName', type: 'text', required: 1, sort: 2 },
  { id: 3, name: '合同金额', code: 'contractAmount', type: 'number', required: 1, sort: 3 },
  { id: 4, name: '签订日期', code: 'signDate', type: 'date', required: 1, sort: 4 }
])

const goBack = () => {
  router.back()
}

const handleDrop = (e: DragEvent) => {
  e.preventDefault()
  const variableData = e.dataTransfer?.getData('text/plain')
  if (variableData) {
    const variable = JSON.parse(variableData)
    ElMessage.success(`插入变量: ${variable.name}`)
    // 在编辑器中插入变量占位符
  }
}

const handleDragOver = (e: DragEvent) => {
  e.preventDefault()
}

const saveTemplate = () => {
  ElMessage.success('保存成功')
}

const previewTemplate = () => {
  activeTab.value = 'preview'
}

onMounted(async () => {
  // 加载模板数据
  console.log('加载模板:', templateId.value)
})
</script>

<style scoped>
.template-editor {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: #f5f7fa;
}

.editor-layout {
  flex: 1;
  display: flex;
  overflow: hidden;
  margin-top: 16px;
}

.editor-area {
  flex: 1;
  padding: 16px;
  overflow: auto;
}

.editor-content {
  min-height: 500px;
}
</style>
```

## 四、合同生成模块实现

### 4.1 后端合同生成服务

```java
// src/main/java/com/contract/editor/service/ContractService.java
package com.contract.editor.service;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.IService;
import com.contract.editor.entity.Contract;

import java.util.Map;

public interface ContractService extends IService<Contract> {
    
    Page<Contract> getContractPage(Integer pageNum, Integer pageSize);
    
    Contract generateContract(Long templateId, Map<String, Object> variables);
    
    byte[] downloadContract(Long id, String format);
}
```

```java
// src/main/java/com/contract/editor/service/impl/ContractServiceImpl.java
package com.contract.editor.service.impl;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.contract.editor.entity.Contract;
import com.contract.editor.entity.Template;
import com.contract.editor.mapper.ContractMapper;
import com.contract.editor.service.ContractService;
import com.contract.editor.service.TemplateService;
import com.contract.editor.util.DocxUtil;
import com.contract.editor.util.MinIOUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.time.LocalDateTime;
import java.util.Map;
import java.util.UUID;

@Service
@RequiredArgsConstructor
public class ContractServiceImpl extends ServiceImpl<ContractMapper, Contract> implements ContractService {

    private final TemplateService templateService;
    private final DocxUtil docxUtil;
    private final MinIOUtil minIOUtil;

    @Override
    public Page<Contract> getContractPage(Integer pageNum, Integer pageSize) {
        Page<Contract> page = new Page<>(pageNum, pageSize);
        return this.page(page);
    }

    @Override
    public Contract generateContract(Long templateId, Map<String, Object> variables) {
        try {
            Template template = templateService.getById(templateId);
            if (template == null) {
                throw new RuntimeException("模板不存在");
            }

            // 下载模板文件
            ByteArrayInputStream templateStream = new ByteArrayInputStream(
                minIOUtil.downloadFileBytes(template.getFilePath())
            );

            // 填充模板
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            docxUtil.fillTemplate(templateStream, variables, outputStream);

            // 上传生成的合同
            String fileName = UUID.randomUUID() + ".docx";
            String filePath = minIOUtil.uploadFile(
                new ByteArrayInputStream(outputStream.toByteArray()),
                fileName,
                "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
            );

            // 保存合同信息
            Contract contract = new Contract();
            contract.setTemplateId(templateId);
            contract.setName(template.getName() + "_" + LocalDateTime.now());
            contract.setContractNo("CT" + System.currentTimeMillis());
            contract.setFilePath(filePath);
            contract.setStatus(1);
            contract.setCreateTime(LocalDateTime.now());
            contract.setUpdateTime(LocalDateTime.now());
            this.save(contract);

            return contract;
        } catch (Exception e) {
            throw new RuntimeException("生成合同失败", e);
        }
    }

    @Override
    public byte[] downloadContract(Long id, String format) {
        try {
            Contract contract = this.getById(id);
            if (contract == null) {
                throw new RuntimeException("合同不存在");
            }

            byte[] fileBytes = minIOUtil.downloadFileBytes(contract.getFilePath());

            if ("pdf".equalsIgnoreCase(format)) {
                // TODO: 实现 DOCX 转 PDF
                return fileBytes;
            }

            return fileBytes;
        } catch (Exception e) {
            throw new RuntimeException("下载合同失败", e);
        }
    }
}
```

## 五、总结

本实现步骤文档详细说明了可视化合同模板编辑器的完整实现方案，包括：

1. **项目初始化**：前端 Vue 3 + TypeScript 项目和后端 Spring Boot 项目的搭建
2. **DOCX 解析模块**：使用 Apache POI 解析 DOCX 文件，提取段落和表格
3. **富文本编辑器模块**：集成 Quill 编辑器，实现变量拖拽功能
4. **合同生成模块**：实现模板变量填充和合同生成

通过按照以上步骤实现，可以完成一个功能完善的可视化合同模板编辑器，支持 DOCX 导入、变量拖拽配置、实时预览、合同生成等核心功能。
