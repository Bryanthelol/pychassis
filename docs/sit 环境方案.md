<!-- ---
hide:
  - footer
--- -->


# sit 环境方案

## 流程

### 前置条件

- feature/feature-a 分支的功能在 test 环境通过验收后

### 主流程

1. 相关人员发起 Pull Request 到 develop 分支
2. 由相关人员进行 Code Review 
3. Code Review 通过后，相关人员合并当前 Pull Request

      - 此时 CI 收到 develop 分支的 push 事件（由 merge pull_request 产生）

          - 跑单元测试（如果有的话）
          - 部署到 sit 服务器（集成测试）
          - 跑代码扫描（如果有的话）

### 备注

- develop 分支禁止主动 Push
- 合代码通过 Pull Request

## drone yaml 配置示例

### 需要准备的 secret

- develop_host
- develop_ssh_key
- gitea_token
- sonar_host
- sonar_token

### 配置代码

```yaml

---

kind: pipeline
type: docker
name: develop

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - develop
  event:
    include:
      - push

clone:
  disable: true

steps:
- name: deploy
  image: appleboy/drone-ssh
  settings:
    host:
      from_secret: develop_host
    username: root
    key:
      from_secret: develop_ssh_key
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

- name: clone
  image: alpine/git
  commands:
    - git clone ${DRONE_GIT_HTTP_URL} .
    - git checkout ${DRONE_BRANCH}

- name: code-analysis
  image: aosapps/drone-sonar-plugin
  volumes:
    - name: sonarqube_cache
      path: /root
  settings:
    sonar_host:
      from_secret: sonar_host
    sonar_token:
      from_secret: sonar_token
    sources: .
    level: DEBUG
    showProfiling: true

volumes:
- name: sonarqube_cache
  host:
    path: /xxx/xxx/sonarqube_cache
```