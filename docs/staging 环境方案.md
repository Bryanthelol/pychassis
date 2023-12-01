<!-- ---
hide:
  - footer
--- -->

# staging 环境方案

## 流程

### 前置条件

- develop 分支的功能在 sit 环境通过验收后

### 主流程

1. 相关人员发起 Pull Request 到 main 分支

      - 此时 CI 收到 main 分支的 pull_request 事件
          - 跑单元测试（如果有的话）
          - 部署到 staging 服务器（预发布环境）

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

---

kind: pipeline
type: docker
name: main-pull-request-staging

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - main
  when:
    event:
      - pull_request
    action:
      include:
        - opened
        - reopened
        - synchronized

clone:
  disable: true

steps:
- name: deploy
  image: appleboy/drone-ssh
  settings:
    host:
      from_secret: staging_host
    username: root
    key:
      from_secret: staging_ssh_key
    port: 22
    script_stop: true
    script:
      - cd /xxx/xxx/xxx
      - git fetch
      - git checkout ${DRONE_BRANCH}
      - git reset --hard ${DRONE_COMMIT}
      - docker-compose down
      - docker-compose up -d --build --force-recreate
      - docker image prune -f

```