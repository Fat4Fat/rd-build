# 构建完整研发内部服务
> 一个完整的研发内部服务（200人以下研发适用）

1. 基础设施搭建。

    - 网络：

      - 网络划分
      - 路由设置
      - 防火墙
      - 企业VPN
      - 海外代理
      - 内网DNS

    - 物理机器

      - 磁盘阵列
      - 小集群管理（1-10台）
      - 容器化

    - 企业软件

      > 核心是组织管理，其他服务是围绕组织管理开展

      - 协同
        - 邮箱
        - IM
        - 网盘
        - wiki
      - 组织管理
        - 通讯录
        - 组织管理
        - 员工统一鉴权（密码+二步认证）
      - OA
        - 财务
        - 行政
        - 人事
        - 审批
        - 报表
      - 研发
        - gitlab
        - 镜像仓库
        - 内网研发中心

    > 部署安装 https://github.com/ifintech/rdbuild/tree/master/build
2. 研发中心

    > 为研发测试提供提升开发效率和保证质量的平台
    > 
    > - 内网资源申请
    > - 上线流程申请
    >
    
    > 详见文档 https://github.com/ifintech/rdcenter

3. 持续迭代研发周期管理

    > 详见文档 https://github.com/ifintech/rdflow

4. 规章制度和研发规范

    > 详见文档 https://github.com/ifintech/rddocs

5. 本地开发环境搭建

    > 详见文档 https://github.com/ifintech/rdbuild/tree/master/local
