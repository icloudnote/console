# AGENTS.md

本文档为 Agent 智能编码助手提供开发规范指导。

## 项目概览

本项目是 iCloudNote 管理控制台的代码仓库，包含三个核心项目：

| 项目 | 路径 | 技术栈 | 端口 |
|------|------|--------|------|
| 前端 | `/Users/zcj/Git/github/icloudnote/react-mix` | UmiJS Max + Ant Design Pro + TypeScript + pnpm | 1080 |
| 后端 | `/Users/zcj/Git/github/icloudnote/go-mix` | Go + Gin + gRPC + MySQL + Redis | 1081 |
| 软件包 | `/Users/zcj/Git/github/icloudnote/gopkg` | Go 工具包集合 | - |

### 项目关系

- **react-mix**: 前端管理界面，调用 go-mix 提供的 OpenAPI
- **go-mix**: 后端服务，提供 REST/gRPC API，依赖 gopkg 中的工具包
- **gopkg**: 公共 Go 工具库（redis, mysql, logger, k8s, github, gitlab 等）

---

## 常用命令

### 前端 (react-mix)

```bash
cd /Users/zcj/Git/github/icloudnote/react-mix

# 安装依赖
pnpm install

# 开发模式（端口 1080）
npm run start:dev

# 无 mock 数据开发
npm run start:no-mock

# 测试环境开发
npm run start:test

# 生产构建
npm run build

# 代码检查（oxlint + prettier）
npm run lint

# 自动修复 lint 错误
npm run lint:fix

# TypeScript 类型检查
npm run tsc

# 代码格式化（oxfmt）
npm run fmt

# 格式化检查
npm run fmt:check

# 运行测试
npm test

# 测试覆盖率
npm run test:coverage

# 更新测试快照
npm run test:update
```

### 后端 (go-mix)

```bash
cd /Users/zcj/Git/github/icloudnote/go-mix

# 代码生成
make wire          # Google Wire 依赖注入生成
make proto         # 生成 protobuf 代码
make openapi       # 生成 OpenAPI 代码

# 代码质量
make format        # 格式化代码（gofumpt + modernize + go vet）
make lint          # 代码检查（golangci-lint）

# 测试
make test          # 运行所有测试
make cover         # 生成覆盖率报告

# 构建运行
make build         # 构建所有服务
make run-openapi-http   # 启动 HTTP 服务（端口 1081）
make run-openapi-worker # 启动 Worker 服务

# 清理
make clean
```

### 软件包 (gopkg)

```bash
cd /Users/zcj/Git/github/icloudnote/gopkg

# 格式化代码
make format

# 运行测试
make test

# 测试覆盖率
make cover

# 更新包依赖
make update

# 安装开发工具
make install
```

---

## 运行单个测试

### 前端 (react-mix)

```bash
# 指定测试文件
npm test -- tests/example.test.ts

# 指定测试函数
npm test -- -t "testName"

# 指定测试文件和函数
npm test -- tests/example.test.ts -t "testName"

# Jest 直接运行
jest tests/example.test.ts -t "testName"
```

### 后端 (go-mix)

```bash
# 指定包
go test ./internal/openapi/service/...

# 指定测试函数
go test -v ./internal/openapi/service/... -run TestName

# 指定测试文件
go test -v ./internal/openapi/service/user_test.go

# 指定包 + 覆盖
go test -cover ./internal/openapi/service/...
```

### 软件包 (gopkg)

```bash
# 进入具体包目录
cd /Users/zcj/Git/github/icloudnote/gopkg/redis

# 运行测试
go test ./...

# 指定测试
go test -v ./... -run TestName
```

---

## 代码风格指南

### 格式化工具

- **前端**: 使用 `oxlint` + `prettier`，通过 `npm run lint` / `npm run fmt` 运行
- **后端**: 使用 `gofumpt` + `modernize`，通过 `make format` 运行

### 导入规范

#### 前端 (TypeScript)

```typescript
// 顺序：React/内置 → 第三方 → 项目内部组件/工具 → 类型
import { useState, useEffect } from 'react';
import { Button, Form, Input } from 'antd';
import { UserOutlined, LockOutlined } from '@ant-design/icons';

import { LoginParams } from '@/types/login';
import { login } from '@/services/user';
import styles from './index.less';
```

#### 后端 (Go)

```go
import (
    "context"
    "fmt"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/spf13/cobra"

    "github.com/icloudnote/go-mix/api/openapi/v1"
    "github.com/icloudnote/go-mix/internal/openapi/model"
    "github.com/icloudnote/gopkg/logger"
)
```

### 命名规范

#### 前端

- 组件文件: `UserList.tsx`, `LoginForm.tsx`（PascalCase）
- 工具函数: `formatDate.ts`, `validateEmail.ts`（kebab-case）
- 样式文件: `index.less`, `userCard.module.less`
- 常量: `MAX_UPLOAD_SIZE`, `API_BASE_URL`（大写 + 下划线）
- 变量/函数: `userList`, `handleSubmit`（camelCase）

#### 后端

- 文件: `user_service.go`, `auth.go`（下划线命名）
- 结构体: `User`, `Project`（大驼峰）
- 变量: `userName`, `projectList`（小驼峰）
- 常量: `TokenPrefix`, `MaxPageSize`（大写 + 下划线）
- 接口: 以 `er` 结尾，如 `Tabler`, `Reader`

### 错误处理

#### 前端

```typescript
// 使用 try-catch 处理异步操作
try {
  await login(params);
  message.success('登录成功');
} catch (error) {
  console.error('登录失败:', error);
  message.error(error.message || '登录失败，请稍后重试');
}
```

#### 后端

```go
// 必须使用 fmt.Errorf + %w 包装错误
if err := db.First(&user).Error; err != nil {
    return nil, fmt.Errorf("获取用户信息失败, %w", err)
}

// 推荐使用 slog 进行结构化日志
slog.Info("用户登录", "user_id", user.ID, "ip", c.ClientIP())
```

### 数据库操作

```go
// 必须使用 WithContext 和 TableName
u.MySQL.DB.Table(model.User{}.TableName()).WithContext(ctx).Where("id=?", id).Find(&user)

// 推荐使用 Scopes 分页
db.Scopes(mysql.Paginate(limit, offset)).Find(&users)
```

### 测试规范

#### 前端

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testingdescribe('Login-library/user-event';

Form', () => {
  it('should render login form', () => {
    render(<LoginForm />);
    expect(screen.getByLabelText('用户名')).toBeInTheDocument();
  });
});
```

#### 后端

```go
import "github.com/stretchr/testify/assert"

func TestUserFavoriteModel(t *testing.T) {
    favorite := model.Favorite{ID: 1, UserID: 1, ProjectID: 2}
    assert.Equal(t, "user_favorites", favorite.TableName())
}
```

### 中文注释规范

1. 请用中文回答问题
2. 请给每个关键节点、比较难懂的代码增加中文注释
3. 避免多层嵌套，提前返回
4. 命名为具有描述性的名称，使代码自解释
5. 避免过长的函数或类，提取函数，将大函数分解为多个小函数
6. 避免重复代码，提取为函数、类或模块

---

## 项目架构

### 前端目录结构 (react-mix)

```
react-mix/
├── src/
│   ├── pages/           # 页面组件，按功能模块组织
│   │   ├── User/       # 用户相关
│   │   ├── Dashboard/  # 仪表板
│   │   ├── CICD/       # CI/CD 流水线
│   │   ├── K8S/        # Kubernetes 资源管理
│   │   └── System/     # 系统管理
│   ├── services/       # API 服务层
│   │   ├── openapi/    # 自动生成的 OpenAPI 服务
│   │   └── swagger/   # Swagger 服务
│   ├── components/     # 可复用 UI 组件
│   ├── models/         # 数据模型
│   ├── utils/          # 工具函数
│   └── global.less     # 全局样式
├── config/             # 配置文件
│   ├── config.ts       # 主配置
│   ├── routes.ts       # 路由配置
│   └── proxy.ts        # 代理配置
└── mock/               # Mock 数据
```

### 后端目录结构 (go-mix)

```
go-mix/
├── api/
│   ├── proto/          # Protobuf 定义
│   └── openapi/v1/     # 生成的 Go 代码
├── cmd/                # 入口程序
│   ├── openapi/        # OpenAPI 服务
│   ├── docker/         # Docker 管理
│   └── helm/           # Helm 工具
├── internal/
│   └── openapi/
│       ├── service/    # 业务逻辑
│       ├── model/      # 数据模型
│       ├── middleware/ # 认证、日志中间件
│       ├── config/     # 配置
│       ├── common/     # 公共工具
│       └── router/     # 路由
└── scripts/             # 构建脚本
```

### 软件包结构 (gopkg)

```
gopkg/
├── aliyun/             # 阿里云服务
├── dockerhub/          # Docker Hub API
├── email/              # 邮件发送
├── github/             # GitHub API
├── gitlab/             # GitLab API
├── helm/               # Helm 工具
├── http/               # HTTP 客户端
├── k8s/                # Kubernetes 工具
├── ldap/               # LDAP 认证
├── logger/             # 日志工具（基于 slog）
├── mysql/              # MySQL 工具
├── nsq/                # NSQ 消息
├── redis/              # Redis 客户端
├── robot/              # 机器人工具
├── tracer/             # 追踪工具
└── utils/              # 通用工具
```

---

## 开发工作流

### Git 提交规范

```bash
# 格式：type(scope): description
git commit -m "feat(api): 添加用户认证接口"
git commit -m "fix(ui): 解决按钮对齐问题"
git commit -m "docs(readme): 更新安装说明"
```

类型说明：
- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 重构
- `test`: 测试相关
- `chore`: 构建或辅助工具的变动

### 分支策略

- 主分支：`main`（受保护，仅通过 PR 合并）
- 开发分支：`dev`（日常开发）
- 功能分支：`feature/功能描述`
- 修复分支：`hotfix/问题描述`

### CI/CD 配置

前端 (react-mix): `.github/workflows/admin.yaml`
- 自动部署到 CloudFlare Pages
- 包含 lint 检查和构建

后端 (go-mix): `.github/workflows/*.yaml`
- 包含多个服务构建流程（docker, github, helm, openapi 等）
- 自动 golangci-lint 检查
- Docker 镜像构建和推送
