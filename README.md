# Linux通过SMTP发送邮件的脚本-20行代码（亲测可用）

在Linux系统运维和自动化脚本开发中，邮件通知功能是必不可少的。之前我分享过基于API的邮件发送方案，今天给大家带来一个更通用的SMTP版本，使用swaks工具实现一键邮件发送，兼容任何支持SMTP的邮箱服务。

## SMTP vs API：为什么选择SMTP方案？

**SMTP方案的优势：**
- 兼容性强，支持所有标准邮箱服务
- 不依赖第三方API服务
- 配置灵活，可切换不同邮件服务商
- 企业内网环境也可使用

## 环境准备：安装swaks工具

swaks是一个强大的SMTP测试工具，我们先安装它：

### Ubuntu/Debian系统：
```bash
sudo apt-get update
sudo apt-get install swaks -y
```

### CentOS/RHEL系统：
```bash
# 启用EPEL仓库
sudo yum install epel-release -y
# 安装swaks
sudo yum install swaks -y
```

### 或者使用包管理器直接安装：
```bash
# 使用curl快速安装
curl -o /usr/local/bin/swaks http://www.jetmore.org/john/code/swaks/files/swaks-20201014.0/swaks
chmod +x /usr/local/bin/swaks
```

## 完整SMTP脚本代码

```bash
#!/bin/bash

# Linux中swaks实现SMTP发送邮件的sh脚本
SMTP_SERVER="smtp.smtpman.cn"
SMTP_PORT="2587"
FROM_EMAIL="info@test111.smtpman.cn"
FROM_PASSWORD="Tid32mfi3ma"
TO_EMAIL="smtpman@163.com"
SUBJECT="SMTP测试邮件"
MESSAGE="这是在Linux中使用bash脚本通过SMTP发送的测试邮件。"

swaks --to "$TO_EMAIL" \
      --from "$FROM_EMAIL" \
      --server "$SMTP_SERVER:$SMTP_PORT" \
      --auth LOGIN \
      --auth-user "$FROM_EMAIL" \
      --auth-password "$FROM_PASSWORD" \
      -tls \
      --body "$MESSAGE" \
      --header "Subject: $SUBJECT"
```

## 详细使用教程

### 1. 创建脚本文件

在系统中创建脚本文件，建议放在系统路径下：

```bash
# 创建脚本文件
sudo vim /usr/local/bin/smtp-mail.sh

# 或者使用cat命令创建
sudo cat > /usr/local/bin/smtp-mail.sh << 'EOF'
#!/bin/bash

# Linux中swaks实现SMTP发送邮件的sh脚本
SMTP_SERVER="smtp.smtpman.cn"
SMTP_PORT="2587"
FROM_EMAIL="info@test111.smtpman.cn"
FROM_PASSWORD="Tid32mfi3ma"
TO_EMAIL="smtpman@163.com"
SUBJECT="SMTP测试邮件"
MESSAGE="这是在Linux中使用bash脚本通过SMTP发送的测试邮件。"

swaks --to "$TO_EMAIL" \
      --from "$FROM_EMAIL" \
      --server "$SMTP_SERVER:$SMTP_PORT" \
      --auth LOGIN \
      --auth-user "$FROM_EMAIL" \
      --auth-password "$FROM_PASSWORD" \
      -tls \
      --body "$MESSAGE" \
      --header "Subject: $SUBJECT"
EOF
```

### 2. 授予执行权限

```bash
# 授予执行权限
sudo chmod +x /usr/local/bin/smtp-mail.sh

# 验证权限
ls -la /usr/local/bin/smtp-mail.sh
```
应该显示：`-rwxr-xr-x 1 root root ...`

### 3. 执行脚本发送邮件

```bash
# 直接执行脚本
/usr/local/bin/smtp-mail.sh

# 或者使用sh命令
sh /usr/local/bin/smtp-mail.sh
```

### 4. 验证发送结果

成功发送后，你会看到类似输出：
```
=== Trying smtp.smtpman.cn:2587...
=== Connected to smtp.smtpman.cn.
...
<** 250 ok:  Message 123456789 accepted
>>> QUIT
...
=== Connection closed with remote host.
```

看到 `250 ok: Message accepted` 就说明邮件发送成功了！

## 脚本参数详解

让我详细解析每个参数的作用：

- **SMTP_SERVER**: SMTP服务器地址
- **SMTP_PORT**: SMTP服务端口（通常587为TLS端口）
- **FROM_EMAIL**: 发件人邮箱地址
- **FROM_PASSWORD**: 发件人邮箱密码或授权码
- **TO_EMAIL**: 收件人邮箱地址
- **SUBJECT**: 邮件主题
- **MESSAGE**: 邮件正文内容

## 支持的主流邮箱服务配置

### 1. 腾讯企业邮箱配置
```bash
SMTP_SERVER="smtp.exmail.qq.com"
SMTP_PORT="465"
FROM_EMAIL="yourname@yourcompany.com"
FROM_PASSWORD="your_password"
# 使用SSL替代TLS
# 将 -tls 改为 -ssl
```

### 2. 阿里企业邮箱配置
```bash
SMTP_SERVER="smtp.mxhichina.com"
SMTP_PORT="465"
FROM_EMAIL="yourname@yourcompany.com"
FROM_PASSWORD="your_password"
```

### 3. Gmail配置
```bash
SMTP_SERVER="smtp.gmail.com"
SMTP_PORT="587"
FROM_EMAIL="yourname@gmail.com"
FROM_PASSWORD="your_app_password"  # 需要使用应用专用密码
```

### 4. 网易163邮箱配置
```bash
SMTP_SERVER="smtp.163.com"
SMTP_PORT="465"
FROM_EMAIL="yourname@163.com"
FROM_PASSWORD="your_authorization_code"  # 需要使用授权码
```

## 高级功能扩展

### 1. 支持命令行动态参数

```bash
#!/bin/bash

# 支持命令行参数的高级版本
TO_EMAIL="${1:-smtpman@163.com}"
SUBJECT="${2:-系统通知}"
MESSAGE="${3:-默认邮件内容}"

SMTP_SERVER="smtp.smtpman.cn"
SMTP_PORT="2587"
FROM_EMAIL="info@test111.smtpman.cn"
FROM_PASSWORD="Tid32mfi3ma"

swaks --to "$TO_EMAIL" \
      --from "$FROM_EMAIL" \
      --server "$SMTP_SERVER:$SMTP_PORT" \
      --auth LOGIN \
      --auth-user "$FROM_EMAIL" \
      --auth-password "$FROM_PASSWORD" \
      -tls \
      --body "$MESSAGE" \
      --header "Subject: $SUBJECT"
```

使用方式：
```bash
# 使用默认参数
/usr/local/bin/smtp-mail.sh

# 自定义收件人和内容
/usr/local/bin/smtp-mail.sh "admin@company.com" "紧急报警" "服务器磁盘空间不足！"
```

### 2. 添加HTML格式邮件支持

```bash
#!/bin/bash

TO_EMAIL="smtpman@163.com"
SUBJECT="HTML格式测试邮件"

# HTML格式内容
MESSAGE_HTML=$(cat << EOF
<html>
<body>
<h2>系统监控报告</h2>
<p><strong>服务器状态：</strong> <span style="color: green;">正常</span></p>
<ul>
<li>CPU使用率: 25%</li>
<li>内存使用率: 60%</li>
<li>磁盘空间: 45%</li>
</ul>
<p><em>报告生成时间: $(date)</em></p>
</body>
</html>
EOF
)

SMTP_SERVER="smtp.smtpman.cn"
SMTP_PORT="2587"
FROM_EMAIL="info@test111.smtpman.cn"
FROM_PASSWORD="Tid32mfi3ma"

swaks --to "$TO_EMAIL" \
      --from "$FROM_EMAIL" \
      --server "$SMTP_SERVER:$SMTP_PORT" \
      --auth LOGIN \
      --auth-user "$FROM_EMAIL" \
      --auth-password "$FROM_PASSWORD" \
      -tls \
      --body "$MESSAGE_HTML" \
      --header "Subject: $SUBJECT" \
      --add-header "Content-Type: text/html"
```

### 3. 添加附件支持

```bash
#!/bin/bash

# 附件版本
TO_EMAIL="smtpman@163.com"
SUBJECT="带附件的测试邮件"
MESSAGE="请查看附件中的系统日志文件。"

SMTP_SERVER="smtp.smtpman.cn"
SMTP_PORT="2587"
FROM_EMAIL="info@test111.smtpman.cn"
FROM_PASSWORD="Tid32mfi3ma"

# 临时创建示例附件
echo "这是系统日志内容..." > /tmp/system.log
echo "$(date): 系统运行正常" >> /tmp/system.log

swaks --to "$TO_EMAIL" \
      --from "$FROM_EMAIL" \
      --server "$SMTP_SERVER:$SMTP_PORT" \
      --auth LOGIN \
      --auth-user "$FROM_EMAIL" \
      --auth-password "$FROM_PASSWORD" \
      -tls \
      --body "$MESSAGE" \
      --header "Subject: $SUBJECT" \
      --attach /tmp/system.log
```

## 实际应用场景

### 1. 系统监控报警脚本

```bash
#!/bin/bash

# 磁盘空间监控
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 90 ]; then
    /usr/local/bin/smtp-mail.sh "admin@company.com" "磁盘空间报警" "根分区使用率: ${DISK_USAGE}%，请及时清理！"
fi
```

### 2. 定时任务执行报告

```bash
#!/bin/bash

# 备份任务报告
BACKUP_LOG="/var/log/backup.log"
/usr/local/bin/backup-script.sh > $BACKUP_LOG 2>&1

if [ $? -eq 0 ]; then
    SUBJECT="备份成功 - $(date +%Y-%m-%d)"
    MESSAGE="系统备份已于 $(date) 成功完成。"
else
    SUBJECT="备份失败 - $(date +%Y-%m-%d)"
    MESSAGE="系统备份失败，请检查日志：$BACKUP_LOG"
fi

/usr/local/bin/smtp-mail.sh "admin@company.com" "$SUBJECT" "$MESSAGE"
```

### 3. 服务状态监控

```bash
#!/bin/bash

# 监控多个服务
SERVICES=("nginx" "mysql" "redis")

for service in "${SERVICES[@]}"; do
    if ! systemctl is-active --quiet $service; then
        /usr/local/bin/smtp-mail.sh "admin@company.com" "服务停止报警" "服务 $service 已停止，请立即处理！"
        systemctl restart $service
    fi
done
```

## 常见问题排查

### Q1: 认证失败怎么办？
A: 检查用户名密码是否正确，某些邮箱需要使用授权码而非登录密码。

### Q2: 连接被拒绝？
A: 检查SMTP服务器地址和端口是否正确，确保防火墙未阻挡。

### Q3: 如何调试发送过程？
A: 添加 `--protocol DEBUG` 参数查看详细通信过程：
```bash
swaks --to "$TO_EMAIL" ... --protocol DEBUG
```

### Q4: 支持SSL连接吗？
A: 将 `-tls` 改为 `-ssl` 即可使用SSL连接。

## 安全建议

1. **保护密码**：不要在脚本中硬编码密码，建议使用环境变量
2. **使用授权码**：对于个人邮箱，建议使用授权码而非真实密码
3. **文件权限**：确保脚本文件权限设置为600，避免密码泄露
4. **日志安全**：避免在日志中记录敏感信息

## 环境变量版本（推荐）

为了避免在脚本中硬编码密码，可以使用环境变量：

```bash
#!/bin/bash

# 从环境变量读取配置
SMTP_SERVER="${SMTP_SERVER:-smtp.smtpman.cn}"
SMTP_PORT="${SMTP_PORT:-2587}"
FROM_EMAIL="${FROM_EMAIL:-info@test111.smtpman.cn}"
FROM_PASSWORD="${FROM_PASSWORD:-Tid32mfi3ma}"
TO_EMAIL="${TO_EMAIL:-smtpman@163.com}"
SUBJECT="${SUBJECT:-SMTP测试邮件}"
MESSAGE="${MESSAGE:-默认邮件内容}"

swaks --to "$TO_EMAIL" \
      --from "$FROM_EMAIL" \
      --server "$SMTP_SERVER:$SMTP_PORT" \
      --auth LOGIN \
      --auth-user "$FROM_EMAIL" \
      --auth-password "$FROM_PASSWORD" \
      -tls \
      --body "$MESSAGE" \
      --header "Subject: $SUBJECT"
```

使用方式：
```bash
# 设置环境变量
export FROM_PASSWORD="your_password"
export TO_EMAIL="recipient@example.com"

# 执行脚本
/usr/local/bin/smtp-mail.sh
```

## 总结

这个20行代码的SMTP邮件发送脚本具有以下优势：

- ✅ **通用性强**：支持所有标准SMTP服务
- ✅ **功能完整**：支持TLS加密、HTML、附件等
- ✅ **易于集成**：可轻松嵌入各种自动化脚本
- ✅ **配置灵活**：支持多邮箱服务商
- ✅ **稳定可靠**：基于成熟的swaks工具

无论是系统监控、定时任务报告还是业务通知，这个脚本都能完美胜任。希望这个SMTP版本的邮件脚本能够帮助大家提高运维效率！

**资源链接**：
- [swaks官网](http://www.jetmore.org/john/code/swaks/)
- [SMTP端口大全](https://www.smtp2go.com/articles/about-smtp-ports/)

如有问题欢迎在评论区交流讨论！
