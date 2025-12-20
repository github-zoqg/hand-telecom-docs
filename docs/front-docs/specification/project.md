---
sidebar_position: 1
sidebar_label: "项目须知"
---

# 项目规范

> 以下规范出自 svn 路径：`02.ERP信创项目\01.项目管理\04项目规范\02内部规范`

## 前端工作计划表

- [2025 年前端工作计划](https://docs.qq.com/sheet/DWXNLVEx6S0xnQ3BX?tab=zdlc7w)

## 仓库地址

- [前端代码仓库：https://code.choerodon.com.cn/cmcc-htcp?filter=xcerp-app](https://code.choerodon.com.cn/cmcc-htcp?filter=xcerp-app)

- 账号密码：与公司邮箱一致

## 测试环境及使用说明

- [测试地址：http://xcerp-app-test.handteledev.com/](http://xcerp-app-test.handteledev.com/)

- 测试环境使用配置

> 1. 下载并登录青浦 vpn（账号需要申请）
> 2. 配置 hosts

```
172.22.5.132     sys.handteledev.com
172.22.5.132     xwbi.handteledev.com
172.22.5.132     api.handteledev.com
172.22.5.132     register.handteledev.com
172.22.5.132     swagger.handteledev.com
172.22.5.132     app.handteledev.com
172.22.5.132     huhu.handteledev.com

172.22.5.132     xcerp-app.handteledev.com
172.22.5.132     xcerp-app-test.handteledev.com

172.22.5.132     xcerp-app-poc.handteledev.com
172.22.5.132     xcerp-poc-test.handteledev.com
172.22.5.132     xcerp-poc.handteledev.com

172.22.5.132     xcerp-app-cac.handteledev.com
172.22.5.132     xcerp-cac-test.handteledev.com
172.22.5.132     xcerp-cac.handteledev.com

172.22.5.132     xcerp-app-glc.handteledev.com
172.22.5.132     xcerp-glc-test.handteledev.com
172.22.5.132     xcerp-glc.handteledev.com

172.22.5.132     xcerp-app-apc.handteledev.com
172.22.5.132     xcerp-apc-test.handteledev.com
172.22.5.132     xcerp-apc.handteledev.com

172.22.5.132     xcerp-app-sac.handteledev.com
172.22.5.132     xcerp-sac-test.handteledev.com
172.22.5.132     xcerp-sac.handteledev.com

172.22.5.132     hand-telecom-docs.handteledev.com
```

## 本地开发指南

项目使用微前端（qiankun），本地开发需启动`主、子`两种服务，[简化版使用文档](../knowledge/qiankun-readme)，[官方文档](https://qiankun.umijs.org/zh/guide)

1. 启动主应用`xcerp-app-front`
2. 启动对应的子应用`xcerp-app-poc-front`或其他
