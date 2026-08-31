---
sidebar_position: 2
sidebar_label: "Dockerfile"
---

# Dockerfile

```dockerfile
# #设置构建镜像☆☆☆☆☆☆☆☆☆☆根据工程构建需要配置带有相应环境的构建用途镜像☆☆☆☆☆☆☆☆☆☆

FROM artifactory.dep.devops.cmit.cloud:20101/xcerp-dev-app-image/node:20.18.1 AS builder
#FROM artifactory.dep.devops.cmit.cloud:20101/xcerp-dev-app-image/node:20.19.5-linuxarm64 AS builder

# 接收构建参数

ARG BRANCH_NAME

# 验证分支名称，只允许 dev、test、uat、prod、master

RUN echo "接收到的分支名称: ${BRANCH_NAME}" && \
    if [ -z "$BRANCH_NAME" ]; then \
 echo "错误: 分支名称不能为空" && exit 1; \
 fi && \
 case "$BRANCH_NAME" in \
        dev|test|uat|prod|master) \
            echo "分支名称验证通过: $BRANCH_NAME" ;; \
        *) \
            echo "错误: 分支名称 '$BRANCH_NAME' 不在允许的范围内" && \
 echo "允许的分支名称: dev、test、uat、prod、master" && \
 exit 1 ;; \
 esac

# 根据分支名称设置对应的 npm registry

RUN echo "根据分支 $BRANCH_NAME 设置对应的 npm registry" && \
    case "$BRANCH_NAME" in \
 dev) \
 NPM_REGISTRY="https://artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe/" ;; \
 test) \
 NPM_REGISTRY="https://artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-test/" ;; \
 uat) \
 NPM_REGISTRY="https://artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-uat/" ;; \
 prod) \
 NPM_REGISTRY="https://artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-prod/" ;; \
 master) \
 NPM_REGISTRY="https://artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-master/" ;; \
 esac && \
 echo "设置的 npm registry: $NPM_REGISTRY" && \
    echo "NPM_REGISTRY=${NPM_REGISTRY}" >> /etc/environment

# #配置工作目录

WORKDIR /webapp

# #将代码仓库拷贝至工作目录

COPY . .
RUN pwd && ls

# #设置前端依赖库

# 配置当前环境nexus的node仓库地址

# 清除 npm 缓存

RUN npm cache clean --force

# 删除 node_modules 和 lock 文件

RUN rm -rf node_modules package-lock.json

# 清除 npm 临时文件

RUN npm cache verify

RUN npm config set registry https://artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/npm

# 若需使用sass_binary_site, 请将下述两行注释打开。若仍出现无法找到等相关报错，请检查是否已经将其已经上传到disturl

#RUN npm config set sass_binary_site=https://artifactory.dep.devops.cmit.cloud:20101/artifactory/npm-remote-nodesass/
#RUN npm config set disturl=https://artifactory.dep.devops.cmit.cloud:20101/artifactory/npm/node/
RUN npm -v && node -v

# #下载依赖包

RUN npm install --legacy-peer-deps
RUN npm run build
#=========================================================================
RUN npm version prerelease --preid=beta --no-git-tag-version

RUN tar -zcvf dist*$(date +%Y%m%d*%H%M%S).tar.gz ./dist ./package.json
RUN ls -lh dist\_\*.tar.gz
RUN . /etc/environment && npm config set registry ${NPM_REGISTRY}

# 根据分支设置对应的认证信息

RUN . /etc/environment && \
 case "$BRANCH_NAME" in \
        dev) \
            echo "//artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe/:_auth=$(echo -n 'jufudsbavyiv:UXrtDP9OLyC2' | base64)" >> $(npm config get userconfig) ;; \
        test) \
            echo "//artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-test/:_auth=$(echo -n 'jufudsbavyiv:UXrtDP9OLyC2' | base64)" >> $(npm config get userconfig) ;; \
        uat) \
            echo "//artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-uat/:_auth=$(echo -n 'jufudsbavyiv:UXrtDP9OLyC2' | base64)" >> $(npm config get userconfig) ;; \
        prod) \
            echo "//artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-prod/:_auth=$(echo -n 'jufudsbavyiv:UXrtDP9OLyC2' | base64)" >> $(npm config get userconfig) ;; \
        master) \
            echo "//artifactory.dep.devops.cmit.cloud:20101/artifactory/api/npm/xcerp-fe-master/:_auth=$(echo -n 'jufudsbavyiv:UXrtDP9OLyC2' | base64)" >> $(npm config get userconfig) ;; \
 esac && \
 echo "设置认证信息完成"

#调试
RUN . /etc/environment && npm whoami --registry=${NPM_REGISTRY}
#推送
RUN . /etc/environment && npm publish dist_*.tar.gz --registry=${NPM_REGISTRY}

#==========================================================================

# #设置基础镜像☆☆☆☆☆☆☆☆☆☆根据工程运行需要配置带有相应环境的运行用途镜像☆☆☆☆☆☆☆☆☆☆

#FROM artifactory.dep.devops.cmit.cloud:20101/cdy_application_factory/nginx:stable-alpine AS final
#FROM artifactory.dep.devops.cmit.cloud:20101/tools/base-images/nginx:alpine AS final

# #将构建镜像中代码的nginx配置文件拷贝至运行镜像nginx目录（默认不开启，nginx有特殊配置注意开启）

#RUN mkdir -p /usr/share/nginx/html/
#COPY --from=builder /webapp/dist/ /usr/share/nginx/dist-pc/dist/

# #☆☆☆☆☆☆☆☆☆☆将构建镜像中打好的dist目录拷贝至运行镜像nginx目录☆☆☆☆☆☆☆☆☆☆

#COPY --from=builder /webapp/dist /usr/share/nginx/html/

#RUN chmod -R 777 /usr/share/nginx/html
```
