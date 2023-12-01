<!-- ---
hide:
  - footer
--- -->

# prod 环境方案

## 流程

### 前置条件

- 测试人员在预发布环境下再次验证功能，团队做上线前的其他准备工作

### 主流程

1. 功能验证通过，相关人员合并 Pull Request

      - （可选项 1）此时 CI 收到 main 分支的 push  事件（由 merge pull_request 产生）

          - CI 将做语义化发布，对 main 分支最新 commit 自动打上版本号并书写 ChangeLog
          - 只在代码托管平台是 github 和 gitlab 时可以使用

      - （可选项 2）在代码托管平台界面对 main 分支的最新 commit 做 Releas 并打上 tag

2. 在选项 1 或者选项 2 执行后
      
      - 此时 CI 收到 main 分支的 tag 事件
          - 构建镜像并推送到镜像仓库
          - 部署到 prod 服务器（生产环境）
              - 通知目标服务器拉取哪个镜像
          		- 如果正在运行容器，则删除
          		- 如果已存在当前镜像，则删除
          		- 目标服务器拉取 registry 的镜像
          		- 把拉取下来的镜像运行成容器
              - 通知 k8s 更新

### 备注

- main 分支禁止主动 Push
- 合代码通过 Pull Request

## drone yaml 配置示例

### 需要准备的 secret

- staging_host
- staging_ssh_key
- gitea_token

### 配置代码

```yaml

# ---
# 如果代码托管平台是 github / gitlab 可以使用语义化发布插件
# kind: pipeline
# type: docker
# name: main-merge-pull-request-prod

# platform:
#   os: linux
#   arch: amd64

# trigger:
#   branch:
#     include:
#       - main
#   event:
#     include:  
#       - push

# clone:
#   disable: true

# steps:
#   - name: semantic-release  
#     image: gtramontina/semantic-release:17.4.3
#     environment:  
#       GITHUB_TOKEN:  
#         from_secret: gitea_token  
#     entrypoint:  
#       - semantic-release
    
---

kind: pipeline
type: docker
name: main-tag-prod

platform:
  os: linux
  arch: amd64

trigger:
  # 备注：如果 event 为 tag，则不能写 branch 条件，否则会导致 tag event 失效
  event:
    include:
      - tag

clone:
  disable: true

steps:
- name: clone-repo
  image: alpine/git
  commands:
    - git clone ${DRONE_GIT_HTTP_URL} .
    - git checkout ${DRONE_BRANCH}

- name: build-and-push-image
  image: plugins/docker
  settings:
    registry:
      from_secret: docker_registry
    username:
      from_secret: docker_username
    password:
      from_secret: docker_password
    repo: gitea.bearcatlog.com/bryant/admin-service
    context: .
    dockerfile: ./Dockerfile
    tags:
    - ${DRONE_TAG}
    purge: true
    compress: true

# TODO 通知 k8s 更新

```

