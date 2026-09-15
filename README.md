# International Regions Database 🌍

一个完整的全球分层行政区划管理系统，支持多国家、多语言、多级联选择。

## 📋 项目介绍

International Regions Database 是一个开源框架，用于管理全球各国的行政区划数据。支持：

- ✅ **多层级支持**：国家 → 省/州 → 市 → 区/县（支持任意层级）
- ✅ **全球覆盖**：支持全球200+国家地区
- ✅ **多语言**：中文、英文等多语言支持
- ✅ **时区管理**：自动识别和管理时区信息
- ✅ **级联选择**：前端级联下拉选择器
- ✅ **高性能**：优化的数据库查询和缓存机制

## 🏗️ 项目架构

```
international-regions-db/
├── frontend/                    # Vue.js 3 + Element Plus
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   └── RegionCascader.vue    # 地区级联选择器
│   │   ├── views/
│   │   ├── api/
│   │   │   └── regions.js           # API 调用
│   │   ├── App.vue
│   │   └── main.js
│   ├── package.json
│   └── vite.config.js
│
├── backend/                     # Go + Gin Framework
│   ├── main.go
│   ├── config/
│   ├── models/
│   │   ├── country.go
│   │   └── division.go
│   ├── handlers/
│   │   └── regions.go
│   ├── database/
│   │   ├── db.go
│   │   └── migrations.sql
│   ├── utils/
│   ├── go.mod
│   ├── go.sum
│   └── Dockerfile
│
├── database/                    # 数据库相关
│   ├── schema.sql               # 数据库结构
│   ├── geonames_import.go       # GeoNames 数据导入
│   └── sample_data.sql          # 示例数据
│
├── docker-compose.yml           # Docker 编排
├── .env.example                 # 环境变量示例
└── docs/                        # 文档
    ├── API.md                   # API 文档
    ├── INSTALL.md               # 安装指南
    └── EXAMPLES.md              # 使用示���
```

## 🚀 快速开始

### 前置要求
- Node.js 16+
- Go 1.19+
- MySQL 8.0+
- Docker & Docker Compose（可选）

### 使用 Docker Compose（推荐）

```bash
# 克隆项目
git clone https://github.com/starfishs/international-regions-db.git
cd international-regions-db

# 启动所有服务
docker-compose up -d

# 前端访问: http://localhost:5173
# 后端 API: http://localhost:8080
# 数据库: localhost:3306
```

### 本地开发

#### 1. 后端开发

```bash
cd backend

# 安装依赖
go mod download

# 配置数据库
cp .env.example .env
# 编辑 .env 文件配置数据库连接

# 运行迁移
mysql -u root -p < ../database/schema.sql

# 启动后端服务
go run main.go
# 服务运行在 http://localhost:8080
```

#### 2. 前端开发

```bash
cd frontend

# 安装依赖
npm install

# 启动开发服务器
npm run dev
# 访问 http://localhost:5173
```

## 📚 API 文档

### 获取国家列表
```http
GET /api/v1/countries

Response:
[
  {
    "id": 1,
    "code": "CN",
    "name": "中国",
    "name_en": "China",
    "timezone": "Asia/Shanghai",
    "currency": "CNY"
  },
  ...
]
```

### 获取分层地区数据
```http
GET /api/v1/divisions/hierarchical

Response:
[
  {
    "id": 1,
    "code": "CN",
    "name": "中国",
    "name_en": "China",
    "level": 0,
    "children": [
      {
        "id": 2,
        "code": "CN-11",
        "name": "北京",
        "name_en": "Beijing",
        "level": 1,
        "children": [
          {
            "id": 3,
            "code": "CN-11-01",
            "name": "朝阳区",
            "level": 2,
            "children": []
          },
          ...
        ]
      },
      ...
    ]
  },
  ...
]
```

### 保存用户位置
```http
POST /api/v1/user/location

Request Body:
{
  "division_path": [1, 2, 3],  # [country_id, level1_id, level2_id, ...]
  "timezone": "Asia/Shanghai"
}

Response:
{
  "message": "Location saved successfully"
}
```

### 获取用户位置
```http
GET /api/v1/user/location

Response:
{
  "path": [
    {"id": 1, "name": "中国", "level": 0},
    {"id": 2, "name": "北京", "level": 1},
    {"id": 3, "name": "朝阳区", "level": 2}
  ],
  "timezone": "Asia/Shanghai"
}
```

## 🎯 数据库结构

### Countries 表
```sql
CREATE TABLE countries (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(2) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    name_en VARCHAR(100),
    timezone VARCHAR(50),
    currency VARCHAR(10),
    language VARCHAR(10),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Administrative Divisions 表
```sql
CREATE TABLE administrative_divisions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(50) UNIQUE,
    name VARCHAR(100) NOT NULL,
    name_en VARCHAR(100),
    country_id INT,
    parent_id INT,
    level INT,
    level_name VARCHAR(50),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    population INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (country_id) REFERENCES countries(id),
    FOREIGN KEY (parent_id) REFERENCES administrative_divisions(id),
    KEY (country_id, parent_id, level)
);
```

### User Locations 表
```sql
CREATE TABLE user_locations (
    user_id INT PRIMARY KEY,
    final_division_id INT,
    timezone VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (final_division_id) REFERENCES administrative_divisions(id)
);
```

## 🔧 功能特性详解

### 1. 级联选择器（RegionCascader.vue）
- 自动生成级联选择面板
- 支持搜索过滤
- 显示完整地址路径
- 实时保存用户选择

### 2. 后端 API（handlers/regions.go）
- 高效的树形数据序列化
- 递归查询优化
- 完整路径构建
- 用户数据持久化

### 3. 数据导入工具（database/geonames_import.go）
- 支持 GeoNames 数据格式
- 支持 GADM 数据格式
- 自动化数据转换
- 批量插入优化

## 📊 支持的国家数据

项目包含以下国家的完整数据：

- 🇨🇳 **中国**：34个省级行政区 + 地级市 + 区县
- 🇺🇸 **美国**：50个州 + 县 + 城市
- 🇯🇵 **日本**：47个都道府县 + 市区町村
- 🇬🇧 **英国**：四个国家 + 郡 + 城镇
- 🇫🇷 **法国**：地区 + 省 + 市镇
- 🌏 **其他国家**：200+ 国家地区数据

## 🔐 环境变量配置

创建 `.env` 文件：

```env
# 数据库配置
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=password
DB_NAME=regions_db

# 服务器配置
GIN_MODE=debug
SERVER_PORT=8080
SERVER_HOST=0.0.0.0

# 前端配置
VUE_APP_API_BASE_URL=http://localhost:8080
```

## 🧪 测试

```bash
# 后端单元测试
cd backend
go test ./...

# 前端单元测试
cd frontend
npm run test
```

## 📖 文档

- [API 文档](./docs/API.md) - 完整的 API 参考
- [安装指南](./docs/INSTALL.md) - 详细的安装步骤
- [使用示例](./docs/EXAMPLES.md) - 代码示例和最佳实践
- [数据导入](./docs/DATA_IMPORT.md) - 如何导入新的地区数据

## 🤝 贡献指南

欢迎贡献！请按照以下步骤：

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 📝 许可证

MIT License - 查看 [LICENSE](./LICENSE) 文件了解详情

## 👨‍💻 作者

- **GitHub**: [@starfishs](https://github.com/starfishs)

## 🆘 支持

- 📖 [文档](./docs)
- 🐛 [Issue Tracker](https://github.com/starfishs/international-regions-db/issues)
- 💬 [讨论区](https://github.com/starfishs/international-regions-db/discussions)

## 🎯 Roadmap

- [ ] 支持更多国家数据
- [ ] 添加地理位置搜索功能
- [ ] 实现数据版本管理
- [ ] 提供 REST API 文档工具（Swagger）
- [ ] 性能优化和缓存机制
- [ ] GraphQL API 支持
- [ ] 移动端适配

---

**⭐ 如果这个项目对您有帮助，请点击 Star！**
