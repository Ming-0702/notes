# CI/CD 学习笔记

## 一、CI/CD 是什么

CI/CD 是现代软件交付流程中的自动化实践。

CI 是 Continuous Integration，持续集成。

CD 可以表示 Continuous Delivery，持续交付；也可以表示 Continuous Deployment，持续部署。

### 1. 持续集成 CI

持续集成的目标是让代码频繁合并，并通过自动化检查尽早发现问题。

常见动作：

- 拉取代码。
- 安装依赖。
- 代码格式检查。
- 静态代码检查。
- 单元测试。
- 构建应用。
- 生成构建产物。

CI 的重点是快速反馈。开发者提交代码后，系统应该尽快告诉你这次提交是否破坏了项目。

### 2. 持续交付 Continuous Delivery

持续交付表示代码经过自动化构建、测试、打包后，随时具备发布到生产环境的能力。

特点：

- 自动完成构建和测试。
- 自动部署到测试环境或预发布环境。
- 发布到生产环境通常需要人工确认。
- 更强调“随时可发布”。

### 3. 持续部署 Continuous Deployment

持续部署是在持续交付基础上更进一步：只要代码通过流水线，就自动发布到生产环境。

特点：

- 自动化程度更高。
- 对测试覆盖、监控、回滚要求更高。
- 适合成熟团队和低风险变更。

### 4. CI/CD 解决的问题

- 减少人工构建和部署错误。
- 缩短从提交代码到上线的时间。
- 提高代码质量。
- 让发布过程可重复、可追踪。
- 让问题更早暴露。
- 支持快速回滚和灰度发布。

## 二、基本流程

一个常见 CI/CD 流程：

```text
开发者提交代码
  -> 触发流水线
  -> 安装依赖
  -> 代码检查
  -> 自动化测试
  -> 构建应用
  -> 构建镜像或打包产物
  -> 推送制品仓库
  -> 部署到测试环境
  -> 人工审批
  -> 部署到生产环境
  -> 监控和回滚
```

不同项目可以裁剪这个流程，但核心思路是：把重复、容易出错、需要一致性的步骤自动化。

## 三、核心概念

### 1. Pipeline

Pipeline 是流水线，表示一次完整的自动化流程。

例如：

```text
lint -> test -> build -> deploy
```

### 2. Stage

Stage 是流水线中的阶段。

常见阶段：

- `lint`
- `test`
- `build`
- `package`
- `deploy`
- `release`

同一个 stage 中的多个 job 通常可以并行执行。

### 3. Job

Job 是流水线中的任务单元。

例如：

- 安装依赖。
- 运行单元测试。
- 构建 Docker 镜像。
- 部署到服务器。

### 4. Step

Step 是 job 内部的一条具体命令或动作。

例如：

```bash
npm ci
npm run lint
npm test
```

### 5. Runner / Agent

Runner 或 Agent 是真正执行流水线任务的机器。

它可以是：

- 平台提供的云主机。
- 自己部署的服务器。
- Kubernetes 中的临时 Pod。
- Docker 容器。

### 6. Artifact

Artifact 是流水线产生的构建产物。

例如：

- 前端 `dist` 目录。
- 后端 jar 包。
- 测试报告。
- 覆盖率报告。
- Docker 镜像。

### 7. Cache

Cache 是缓存，用于加速流水线。

常见缓存内容：

- npm 依赖缓存。
- Maven 本地仓库。
- Gradle 缓存。
- pip 缓存。
- Docker build cache。

Cache 是为了加速，不应该作为唯一可靠产物来源。

### 8. Secret

Secret 是流水线中的敏感配置。

例如：

- SSH 私钥。
- API Token。
- 数据库密码。
- 镜像仓库密码。
- 云服务访问密钥。

Secret 应该放在 CI/CD 平台的密钥管理中，不要写进代码仓库。

### 9. Environment

Environment 表示部署环境。

常见环境：

- `dev`：开发环境。
- `test`：测试环境。
- `staging`：预发布环境。
- `production`：生产环境。

不同环境应该使用不同配置、不同权限和不同审批规则。

## 四、流水线阶段设计

### 1. 常见阶段

| 阶段 | 作用 |
| --- | --- |
| Checkout | 拉取代码 |
| Install | 安装依赖 |
| Lint | 检查代码风格和潜在错误 |
| Test | 运行自动化测试 |
| Build | 构建应用 |
| Package | 打包制品或镜像 |
| Scan | 安全扫描、依赖扫描、镜像扫描 |
| Deploy | 部署到目标环境 |
| Verify | 部署后健康检查 |
| Notify | 通知结果 |

### 2. 设计原则

- 越快的检查越靠前。
- 失败后尽早停止，避免浪费资源。
- 依赖安装要可重复。
- 构建产物应该可追踪。
- 部署步骤要支持回滚。
- 生产部署需要权限控制。
- 密钥不能出现在日志中。
- 流水线配置也要纳入版本控制。

### 3. 快速反馈和完整验证

可以把检查分成两类：

| 类型 | 目标 | 示例 |
| --- | --- | --- |
| 快速检查 | 几分钟内反馈基础问题 | lint、类型检查、单元测试 |
| 完整检查 | 更全面但耗时较长 | 集成测试、端到端测试、安全扫描 |

Pull Request 阶段通常追求快速反馈。合并到主分支后可以运行更完整的检查。

## 五、触发方式

常见触发方式：

- push 到分支。
- 创建 Pull Request / Merge Request。
- 合并到主分支。
- 创建 tag。
- 手动触发。
- 定时任务。
- 外部系统通过 webhook 触发。

示例：

```text
push feature 分支 -> 运行 lint 和 test
pull request -> 运行 lint、test、build
merge main -> 构建镜像并部署测试环境
tag v1.0.0 -> 发布生产版本
```

## 六、分支和发布策略

### 1. Feature Branch

每个功能使用独立分支开发，然后通过 PR/MR 合并。

优点：

- 方便 code review。
- 不直接污染主分支。
- 适合大多数团队。

注意：

- 分支存在时间不要太长。
- 经常同步主分支，减少冲突。
- 合并前必须通过自动化检查。

### 2. Trunk Based Development

开发者频繁向主干合并小改动，配合功能开关控制未完成能力。

优点：

- 集成更频繁。
- 分支分叉时间短。
- 发布节奏快。

要求：

- 测试自动化程度高。
- 主分支保护严格。
- 功能开关管理可靠。

### 3. Git Flow

常见分支：

- `main` 或 `master`：生产版本。
- `develop`：开发主线。
- `feature/*`：功能分支。
- `release/*`：发布分支。
- `hotfix/*`：紧急修复。

Git Flow 流程清晰，但分支较多，适合发布周期相对固定的项目。

### 4. Tag 发布

生产发布常用 tag 标记版本：

```bash
git tag v1.0.0
git push origin v1.0.0
```

CI/CD 可以配置为 tag 触发发布。

版本号建议使用语义化版本：

```text
MAJOR.MINOR.PATCH
```

例如：

```text
v2.3.1
```

## 七、GitHub Actions 示例

GitHub Actions 的配置通常放在：

```text
.github/workflows/*.yml
```

### 1. Node.js CI 示例

```yaml
name: Node CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

### 2. 构建并推送 Docker 镜像

```yaml
name: Build Docker Image

on:
  push:
    tags:
      - "v*"

jobs:
  docker:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/owner/app:${{ github.ref_name }}
```

这里的 `${{ github.ref_name }}` 在 tag 触发时就是 tag 名称，例如 `v1.0.0`。

## 八、GitLab CI 示例

GitLab CI 配置文件通常是：

```text
.gitlab-ci.yml
```

示例：

```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - npm run lint
    - npm test

build:
  stage: build
  image: node:20-alpine
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy_staging:
  stage: deploy
  script:
    - echo "deploy to staging"
  only:
    - main
```

说明：

- `stages` 定义阶段顺序。
- `image` 指定 job 运行环境。
- `script` 是具体命令。
- `artifacts` 保存构建产物。
- `only` 控制 job 触发条件。

## 九、Jenkins 基本概念

Jenkins 是常见的自建 CI/CD 工具。

核心概念：

- Controller：Jenkins 主节点，负责调度和管理。
- Agent：执行任务的节点。
- Job：任务。
- Pipeline：流水线。
- Jenkinsfile：流水线配置文件。

Declarative Pipeline 示例：

```groovy
pipeline {
  agent any

  stages {
    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build'
      }
    }
  }
}
```

Jenkins 灵活度高，但需要自己维护插件、权限、节点、凭据和升级。

## 十、制品管理

CI/CD 中不要只关注“把代码部署上去”，还要关注产物是否可追踪、可复用、可回滚。

常见制品：

- Docker 镜像。
- jar / war 包。
- npm 包。
- Python wheel 包。
- 前端静态资源。
- Helm chart。

制品仓库：

- Docker Registry。
- GitHub Packages。
- GitLab Package Registry。
- Nexus。
- Artifactory。
- Harbor。

好的制品应该具备：

- 明确版本号。
- 来源 commit 可追踪。
- 构建时间可追踪。
- 依赖信息可追踪。
- 可以被重复部署。

## 十一、Docker 在 CI/CD 中的使用

### 1. 常见流程

```text
拉取代码
  -> 运行测试
  -> docker build 构建镜像
  -> docker push 推送镜像仓库
  -> 部署平台拉取镜像
  -> 启动新版本容器
```

### 2. 镜像标签建议

不要只使用 `latest`。

推荐同时打多个标签：

```text
app:v1.2.0
app:git-a1b2c3d
app:main
```

生产部署最好使用：

- 版本号 tag。
- Git commit SHA tag。
- 镜像 digest。

这样方便定位和回滚。

### 3. 镜像构建注意点

- CI 中构建镜像要使用确定版本的基础镜像。
- 不要把 `.env`、私钥、Token 放进镜像。
- 使用多阶段构建。
- 使用 `.dockerignore` 减少构建上下文。
- 对镜像进行安全扫描。
- 推送镜像前确保测试已通过。

## 十二、部署策略

### 1. 直接部署

停止旧版本，启动新版本。

优点：

- 简单。
- 成本低。

缺点：

- 可能有短暂停机。
- 出问题影响范围大。

### 2. 滚动发布

逐批替换旧实例。

优点：

- 不需要同时启动完整两套环境。
- 可以减少停机时间。

缺点：

- 发布期间新旧版本会同时存在。
- 需要兼容数据库和接口变更。

### 3. 蓝绿发布

准备两套环境：

- 蓝色：当前生产环境。
- 绿色：新版本环境。

验证绿色环境无问题后，把流量切到绿色。

优点：

- 回滚快。
- 发布风险较低。

缺点：

- 资源成本较高。
- 数据库变更仍需谨慎。

### 4. 金丝雀发布

先让少量用户访问新版本，观察指标正常后逐步扩大流量。

优点：

- 风险可控。
- 适合高流量线上服务。

要求：

- 有流量切分能力。
- 有监控和告警。
- 有快速回滚机制。

### 5. 功能开关

功能开关可以让代码先上线，但功能暂时关闭。

用途：

- 控制新功能灰度。
- 快速关闭异常功能。
- 支持 trunk based development。

注意：

- 功能开关要有清理机制。
- 不要让过期开关长期堆积。

## 十三、数据库变更

数据库变更是 CI/CD 中最容易出事故的部分。

建议：

- 数据库迁移脚本纳入版本控制。
- 每次变更可以重复执行或有明确版本记录。
- 先做兼容性变更，再切换代码。
- 避免上线时直接删除字段或改字段含义。
- 大表变更要评估锁表和性能影响。
- 发布前备份关键数据。

兼容发布示例：

```text
第 1 次发布：新增字段，旧代码不使用
第 2 次发布：新代码同时兼容新旧字段
第 3 次发布：数据迁移完成后移除旧逻辑
第 4 次发布：确认无风险后删除旧字段
```

## 十四、质量门禁

质量门禁是决定代码能否进入下一阶段的规则。

常见门禁：

- 单元测试必须通过。
- 构建必须成功。
- lint 无错误。
- 测试覆盖率不低于阈值。
- 依赖漏洞不能超过指定等级。
- 镜像扫描不能有高危漏洞。
- PR 必须经过 review。
- 主分支必须保护。
- 生产部署必须人工审批。

门禁不是越多越好，关键是能覆盖主要风险，并且不会让反馈速度变得不可接受。

## 十五、安全实践

### 1. 密钥管理

- 使用 CI/CD 平台的 Secret 功能。
- 不要把密钥写进仓库。
- 不要在日志中打印密钥。
- 不要把生产密钥暴露给 PR 流水线。
- 定期轮换密钥。
- 给不同环境使用不同密钥。

### 2. 权限最小化

流水线账号只应该拥有完成任务所需的最小权限。

例如：

- 构建流水线只需要读代码和写镜像仓库。
- 部署测试环境不应该拥有生产环境权限。
- 生产发布需要更严格审批。

### 3. 依赖和镜像安全

常见检查：

- 依赖漏洞扫描。
- 静态代码扫描。
- Secret 扫描。
- Docker 镜像漏洞扫描。
- License 检查。
- SBOM 生成。

### 4. 供应链安全

注意点：

- 固定 action、插件、基础镜像版本。
- 使用可信来源的第三方 action。
- 不在 PR 中运行不可信部署脚本。
- 保护主分支和 tag。
- 对发布制品进行签名。

## 十六、监控和回滚

发布不是流水线结束就完成了。上线后还要观察系统是否健康。

常见监控指标：

- 错误率。
- 请求延迟。
- CPU、内存、磁盘。
- 容器重启次数。
- 日志异常。
- 业务指标。
- 队列堆积。
- 数据库连接数。

回滚方式：

- 回滚到上一个 Docker 镜像版本。
- 回滚 Kubernetes Deployment。
- 切回蓝绿发布中的旧环境。
- 关闭功能开关。
- 回滚数据库变更。

数据库回滚通常最复杂，因此数据库变更应尽量设计为向前兼容。

## 十七、常见问题

### 1. 流水线很慢

优化方向：

- 使用依赖缓存。
- 并行执行独立任务。
- 拆分快速检查和完整检查。
- 减少不必要的构建。
- 优化 Dockerfile 缓存层。
- 使用更合适的 runner 规格。

### 2. 测试偶发失败

常见原因：

- 测试依赖时间、随机数或外部服务。
- 并发测试互相影响。
- 测试数据没有隔离。
- 网络请求不稳定。
- 等待条件不准确。

处理方式：

- 修复 flaky test，不要长期忽略。
- 给集成测试提供稳定测试环境。
- 使用 mock 或 test container。
- 明确超时和重试策略。

### 3. 本地正常，CI 失败

检查方向：

- Node、Java、Python 等运行时版本是否一致。
- 操作系统差异。
- 环境变量是否缺失。
- 依赖锁文件是否提交。
- 文件路径大小写问题。
- 测试是否依赖本地文件或本地服务。

### 4. 部署成功但服务不可用

检查方向：

- 应用是否真的启动完成。
- 健康检查是否通过。
- 配置和密钥是否正确。
- 数据库连接是否正常。
- 端口、域名、负载均衡是否正确。
- 新版本是否兼容旧数据。
- 日志中是否有异常。

## 十八、学习重点

- 理解 CI、持续交付、持续部署的区别。
- 知道 pipeline、stage、job、artifact、cache、secret 的含义。
- 会写基础 GitHub Actions 或 GitLab CI 配置。
- 会把测试、构建、打包、部署串成流水线。
- 明白密钥不能写进仓库。
- 知道生产发布需要回滚方案。
- 知道不同部署策略的优缺点。
- 能根据日志和流水线结果定位失败原因。

## 十九、CI/CD 检查清单

提交代码前：

- 本地测试通过。
- 依赖锁文件已提交。
- 没有提交密钥和敏感配置。
- 变更范围清晰。

合并前：

- CI 通过。
- Code review 完成。
- 测试覆盖主要逻辑。
- 数据库变更已评估。

发布前：

- 构建产物可追踪。
- 镜像或制品版本明确。
- 配置和密钥已准备。
- 回滚方案明确。
- 监控和告警可用。

发布后：

- 查看应用日志。
- 查看错误率和延迟。
- 验证核心业务流程。
- 观察资源占用。
- 记录版本和发布时间。
