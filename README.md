# 纵横智慧云图库 - 企业级智能协同云图库平台

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Vue](https://img.shields.io/badge/Vue-3.5-green.svg)](https://vuejs.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.7-green.svg)](https://spring.io/projects/spring-boot)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue.svg)](https://www.mysql.com/)

> 基于 Vue 3 + Spring Boot + COS + WebSocket 构建的企业级智能协同云图库平台，集图片存储、管理、检索、协同编辑于一体。

## 📖 项目介绍

智慧图库是一个功能完善的现代化图片管理平台，支持多场景应用，为个人用户、团队和企业提供专业的图片存储、管理和协作解决方案。

### 🌟 核心特性

- **🔐 多级权限管理**: 支持公开图库、私人空间、团队协作等多种权限模式
- **📱 智能图片处理**: AI自动标签、智能分类、相似图片识别
- **⚡ 高性能架构**: 分库分表、Redis缓存、WebSocket实时通信
- **🎨 图片编辑**: 在线裁剪、颜色调整、实时协同编辑
- **📊 数据分析**: 存储统计、使用分析、性能监控
- **🚀 企业级部署**: 支持集群部署、负载均衡、高可用架构

## 🛠️ 技术架构

### 后端技术栈
- **核心框架**: Spring Boot 2.7.6
- **开发语言**: Java 11
- **数据库**: MySQL 8.0 + MyBatis-Plus 3.5.9
- **缓存方案**: Redis分布式缓存 + Caffeine本地缓存
- **文件存储**: 腾讯云COS对象存储
- **分库分表**: ShardingSphere 5.2.0
- **权限控制**: Sa-Token 1.39.0
- **实时通信**: WebSocket + Disruptor高性能队列
- **API文档**: Knife4j + OpenAPI
- **数据采集**: Jsoup网页解析
- **工具库**: Hutool 5.8.26

### 前端技术栈
- **核心框架**: Vue 3.5 + TypeScript
- **构建工具**: Vite 6.2
- **UI组件库**: Ant Design Vue 4.2
- **HTTP客户端**: Axios 1.8
- **状态管理**: Pinia 3.0
- **路由管理**: Vue Router 4.5
- **数据可视化**: ECharts 5.6 + 词云图
- **图片处理**: Vue-Cropper图片裁剪
- **代码规范**: ESLint + Prettier
- **API生成**: OpenAPI自动生成

## 📁 项目结构

```
zh-picture/
├── zh-picture-backend/          # 后端服务
│   ├── src/main/java/
│   │   └── com/cheng/zhpicturebackend/
│   │       ├── annotation/      # 自定义注解
│   │       ├── aop/            # 切面编程
│   │       ├── api/            # 外部API集成
│   │       ├── common/         # 通用类
│   │       ├── config/         # 配置类
│   │       ├── constant/       # 常量定义
│   │       ├── controller/     # 控制器层
│   │       ├── exception/      # 异常处理
│   │       ├── manager/        # 业务管理层
│   │       ├── mapper/         # 数据访问层
│   │       ├── model/          # 数据模型
│   │       ├── service/        # 业务逻辑层
│   │       └── utils/          # 工具类
│   ├── src/main/resources/
│   │   ├── application.yml     # 主配置文件
│   │   ├── application-local.yml  # 本地环境配置
│   │   ├── application-test.yml   # 测试环境配置
│   │   ├── mapper/            # MyBatis映射文件
│   │   └── biz/               # 业务配置
│   ├── sql/
│   │   └── create_table_sql.sql  # 数据库建表脚本
│   ├── httpTest/              # HTTP接口测试
│   └── pom.xml                # Maven依赖配置
│
└── zh-picture-frontend/         # 前端应用
    ├── src/
    │   ├── access/             # 权限控制
    │   ├── api/                # API接口
    │   ├── components/         # 通用组件
    │   │   ├── analyze/        # 数据分析组件
    │   │   └── icons/          # 图标组件
    │   ├── constants/          # 常量定义
    │   ├── layouts/            # 布局组件
    │   ├── pages/              # 页面组件
    │   │   ├── admin/          # 管理员页面
    │   │   └── user/           # 用户页面
    │   ├── router/             # 路由配置
    │   ├── stores/             # 状态管理
    │   ├── utils/              # 工具函数
    │   └── views/              # 视图组件
    ├── package.json            # 项目依赖
    ├── vite.config.ts          # Vite配置
    ├── tsconfig.json           # TypeScript配置
    └── openapi.config.js       # OpenAPI配置
```

## 🚀 快速开始

### 环境要求

- **Java**: JDK 11+
- **Node.js**: 18.0+
- **数据库**: MySQL 8.0+
- **缓存**: Redis 7.0+
- **构建工具**: Maven 3.6+, npm/yarn

### 🗄️ 数据库初始化

1. 创建数据库
```sql
CREATE DATABASE zh_picture CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

2. 执行建表脚本
```bash
mysql -u root -p zh_picture < zh-picture-backend/sql/create_table_sql.sql
```

### ⚙️ 后端部署

1. **修改配置文件**
```yaml
# zh-picture-backend/src/main/resources/application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/zh_picture
    username: your_username
    password: your_password

# 配置腾讯云COS（可选）
cos:
  client:
    host: your_cos_host
    secretId: your_secret_id
    secretKey: your_secret_key
    region: your_region
    bucket: your_bucket
```

2. **启动后端服务**
```bash
cd zh-picture-backend
mvn clean install
mvn spring-boot:run
```

服务启动后访问: http://localhost:8123/api

API文档地址: http://localhost:8123/api/doc.html

### 🎨 前端部署

1. **安装依赖**
```bash
cd zh-picture-frontend
npm install
```

2. **生成API代码（可选）**
```bash
npm run openapi
```

3. **启动开发服务器**
```bash
npm run dev
```

前端访问地址: http://localhost:5173

### 📦 生产环境部署

**后端打包**
```bash
cd zh-picture-backend
mvn clean package -Dmaven.test.skip=true
java -jar target/zh-picture-backend-0.0.1-SNAPSHOT.jar
```

**前端打包**
```bash
cd zh-picture-frontend
npm run build
# 将dist目录部署到nginx或其他Web服务器
```

## 🌐 功能模块

### 🎯 核心功能

| 功能模块 | 描述 | 技术实现 |
|---------|------|----------|
| **用户管理** | 注册登录、权限控制 | Sa-Token + Redis |
| **图片上传** | 批量上传、拖拽上传 | COS对象存储 + 前端组件 |
| **图片管理** | 分类、标签、检索 | MySQL + 全文索引 |
| **空间管理** | 私人/团队空间 | 多租户架构 |
| **实时协作** | 协同编辑图片 | WebSocket + Disruptor |
| **数据分析** | 使用统计、趋势分析 | ECharts可视化 |

### 📱 应用场景

- **🖼️ 个人图库**: 个人照片管理、作品集展示
- **👥 团队协作**: 企业素材库、项目资源共享
- **🏢 企业应用**: 品牌资产管理、营销素材库
- **🎨 设计平台**: 设计素材网站、模板库
- **📸 社交分享**: 图片社区、表情包网站

## 📊 系统特性

### 🔒 安全特性
- JWT token认证
- RBAC权限控制  
- 图片审核机制
- 访问频率限制
- 敏感信息脱敏

### ⚡ 性能特性
- Redis多级缓存
- 分库分表设计
- CDN加速分发
- 图片压缩优化
- 懒加载机制

### 🎨 用户体验
- 响应式设计
- 拖拽上传
- 实时预览
- 快捷键支持
- 暗色主题

## 🔧 开发指南

### 📝 代码规范

**后端规范**
- 遵循阿里巴巴Java开发手册
- 使用Lombok简化代码
- 统一异常处理机制
- MyBatis-Plus代码生成

**前端规范**
- TypeScript严格模式
- ESLint + Prettier代码格式化
- Vue 3 Composition API
- 组件库统一UI风格

### 🧪 测试

**后端测试**
```bash
cd zh-picture-backend
mvn test
```

**前端测试**
```bash
cd zh-picture-frontend
npm run lint
npm run type-check
```

### 📖 API文档

启动后端服务后访问 Knife4j 文档:
- 开发环境: http://localhost:8123/api/doc.html
- 测试环境: http://your-domain/api/doc.html

## 🎯 路线图

### 近期计划
- [ ] 移动端适配优化
- [ ] 图片AI智能标注
- [ ] 视频文件支持
- [ ] 数据导入导出

### 长期规划
- [ ] 微服务架构重构
- [ ] 多云存储支持
- [ ] 国际化多语言
- [ ] 插件生态系统

## 🤝 贡献指南

我们欢迎社区贡献！请遵循以下步骤：

1. **Fork** 项目到您的GitHub
2. **创建分支**: `git checkout -b feature/amazing-feature`
3. **提交更改**: `git commit -m 'Add amazing feature'`
4. **推送分支**: `git push origin feature/amazing-feature`
5. **创建PR**: 提交Pull Request

### 📋 贡献类型
- 🐛 Bug修复
- ✨ 新功能开发
- 📝 文档完善
- 🎨 UI/UX改进
- ⚡ 性能优化

## 📄 许可证

本项目采用 [MIT License](LICENSE) 开源协议。

## 🙏 致谢

感谢以下开源项目和服务商的支持：

- [Vue.js](https://vuejs.org/) - 前端框架
- [Spring Boot](https://spring.io/projects/spring-boot) - 后端框架
- [Ant Design Vue](https://antdv.com/) - UI组件库
- [MyBatis-Plus](https://baomidou.com/) - 数据库框架
- [腾讯云COS](https://cloud.tencent.com/product/cos) - 对象存储
- [Sa-Token](https://sa-token.cc/) - 权限框架

## 📞 联系我们

- **前端仓库**: [zh-picture-frontend](https://github.com/Origin-Xin/zh-picture-frontend)
- **后端仓库**: [zh-picture-backend](https://github.com/Origin-Xin/zh-picture-backend)
- **问题反馈**: [Frontend Issues](https://github.com/Origin-Xin/zh-picture-frontend/issues) | [Backend Issues](https://github.com/Origin-Xin/zh-picture-backend/issues)
- **邮箱**: support@zhpicture.com

---

⭐ **如果这个项目对您有帮助，请给我们一个Star支持！**

<div align="center">
  <img src="https://your-domain.com/logo.png" alt="智慧图库" width="200"/>
  
  **智慧图库 - 让图片管理更智能**
</div>
