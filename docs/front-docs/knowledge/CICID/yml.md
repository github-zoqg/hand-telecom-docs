---
sidebar_position: 1
sidebar_label: ".yml"
---

# YAML

```yaml
#变量介绍☆☆☆☆☆
#${{env.PROJECT_PATH}}:代码根目录
#${{env.TAG}}:$(date +%s):从 1970 年 1 月 1 日 00:00:00 UTC 到目前为止的秒数（时间戳）

#名称：Docker镜像持续交付
name: Docker Image CI

#获取代码
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

#开始工作
jobs:
  #获取镜像
  build:
    #镜像名称：cmit-docker-ci
    name: cmit-docker-vue
    #镜像版本：ubuntu-latest
    runs-on: ubuntu-latest
    #执行步骤
    steps:
      #切换分支
      - uses: actions/checkout@v2
      #设置镜像仓库☆☆☆☆☆☆☆☆☆☆修改cicd-test/cicd_test为当前工程使用的'镜像库/工程名'☆☆☆☆☆☆☆☆☆☆
      - name: set image repository
        run: |
          # 获取当前仓库名称
          REPO_NAME=$(basename "${{ github.repository }}")

          # 清理分支名，移除特殊字符，只保留字母、数字、连字符
          BRANCH_NAME="${{ github.ref_name }}"
          CLEAN_BRANCH=$(echo "$BRANCH_NAME" | sed 's/[^a-zA-Z0-9-]/-/g' | tr '[:upper:]' '[:lower:]')

          # 构建包含分支信息的镜像仓库名
          IMAGE_REPO="artifactory.dep.devops.cmit.cloud:20101/xcerp-dev-app-image/${REPO_NAME}/${REPO_NAME}-${{env.TAG}}-${CLEAN_BRANCH}"

          # 输出到环境变量和文件
          echo "IMAGE_REPO=${IMAGE_REPO}" | tee -a $GITHUB_ENV ${{env.PROJECT_PATH}}/env
          echo "BRANCH_NAME=${CLEAN_BRANCH}" | tee -a $GITHUB_ENV ${{env.PROJECT_PATH}}/env
          echo "REPO_NAME=${REPO_NAME}" | tee -a $GITHUB_ENV ${{env.PROJECT_PATH}}/env

          # 验证设置结果
          echo "当前仓库: ${REPO_NAME}"
          echo "设置的镜像仓库: ${IMAGE_REPO}"
          echo "当前分支: ${CLEAN_BRANCH}"
      #登录镜像库
      - name: docker login
        run: docker login artifactory.dep.devops.cmit.cloud:20101 -u ${{env.USER_NAME}} -p ${{env.PASSWORD}}
      #设置构建的包类型
      - name: set buildmode
        # default: stage
        if: env.BUILD_MODE == null
        run: echo "BUILD_MODE=stage" | tee -a $GITHUB_ENV ${{env.PROJECT_PATH}}/env
      #制作镜像☆☆☆☆☆☆☆☆☆☆进入代码仓库根目录制作镜像Dockerfile需要根据项目工程进行修改☆☆☆☆☆☆☆☆☆☆
      - name: Build the Docker image
        run: |
          cd ${{env.PROJECT_PATH}} && \
          docker build . \
            --file Dockerfile \
            --tag ${{env.IMAGE_REPO}} \
            --build-arg BUILD_MODE=${{env.BUILD_MODE}} \
            --build-arg BRANCH_NAME=${{env.BRANCH_NAME}}

      #删除镜像
      - name: docker remove
        run: docker rmi ${{env.IMAGE_REPO}}
```
