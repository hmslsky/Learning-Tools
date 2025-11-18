
# 一、概述与价值

## 1.1 简介

**Nginx（Engine-X）** 是一个高性能的 **Web Server + 反向代理 + 负载均衡 + 缓存加速平台**，由 Igor Sysoev 于 2004 年开发，具有超高并发、高稳定性、低资源占用和模块化的特性。

其核心使命：

| 目标                 | 说明                     |
| ------------------ | ---------------------- |
| 处理高并发、高吞吐的 HTTP 请求 | 可稳定支撑数十万并发连接           |
| 承载现代微服务架构流量        | 提供反向代理与负载均衡            |
| 提供内容缓存与性能优化        | 支持静态与动态混合服务            |
| 提供安全与访问控制能力        | WAF、限流、IP 控制、HTTPS/TLS |

典型使用人群：

* Web 后端开发工程师、API 设计者
* DevOps / 运维 / 系统管理员
* 云原生架构师、微服务架构设计者
* 网关组件使用者（如 K8s Ingress）

---

## 1.2 核心优势

| 优势         | 说明                   | 与竞品对比 (Apache, HAProxy, Caddy) |
| ---------- | -------------------- | ------------------------------ |
| 高并发能力强     | 使用 **异步、事件驱动架构**     | > Apache 多线程模型                 |
| 低内存占用      | 单进程 + worker 模型，极低成本 | Caddy 较高，Apache 较高             |
| 模块化设计      | 标准+第三方模块丰富           | 与 HAProxy 相当                   |
| 反向代理与负载均衡强 | 支持多种 LB 算法、健康检查      | 比 Caddy 更专业                    |
| 静态资源处理极快   | 可作为专业 CDN 边缘加速器      | 强于多数同类                         |
| 配置灵活可扩展    | 大量指令、变量、高自定义性        | Caddy 更简单但弱配置                  |
| 超强生态       | K8s、CDN、云厂商默认支持      | 行业标准级别                         |

一句话总结：

> **Nginx 是现代互联网流量的基础设施，适用于从简单站点到全球级分布式系统。**

---

## 1.3 典型应用场景（问题 → 对应解决方案）

| 业务需求 / 问题              | Nginx 解决方案           | 模块                   |
| ---------------------- | -------------------- | -------------------- |
| 静态网站托管（HTML、图片）        | 高速静态资源服务             | core                 |
| Node/Python/Java 微服务部署 | 反向代理 + 负载均衡          | proxy                |
| 多域名 & HTTPS 网关         | SSL/TLS + SNI + 证书管理 | stream/http/ssl      |
| API 限流防刷               | rate limiting        | limit_req/limit_conn |
| CDN 缓存 & 回源            | 缓存代理                 | proxy_cache          |
| 灰度发布 & 蓝绿部署            | upstream 负载策略        | upstream             |
| WebSocket 转发           | 反向代理支持升级协议           | proxy                |
| 边缘安全防护                 | WAF / 黑白名单 / TLS     | http_access          |

---

## 1.4 文档阅读指南

| 章节   | 适合读者    | 内容侧重       |
| ---- | ------- | ---------- |
| 第一部分 | 初学者     | 理解技术定位     |
| 第二部分 | 新手实践者   | 快速部署运行     |
| 第三部分 | 开发 & 运维 | 核心配置与功能    |
| 第四部分 | 生产环境用户  | 最佳实践 & 高可用 |
| 第五部分 | 全部用户    | 学习路线       |

---

# 二、基础环境与配置

## 2.1 准备工作

### 支持系统

* Linux（主流生产首选）
* macOS（学习与开发）
* Windows（不推荐用于生产）
* Docker / K8s（现代主流部署方式）

### 推荐最低配置

| 环境  | 配置     |
| --- | ------ |
| CPU | 1核+    |
| RAM | 512MB+ |
| 磁盘  | 20MB+  |

### 安装方式

```bash
# Ubuntu
sudo apt update && sudo apt install nginx -y

# CentOS / Rocky / AlmaLinux
sudo yum install epel-release -y
sudo yum install nginx -y

# Docker
docker run -d -p 80:80 -p 443:443 nginx
```

---

## 2.2 基本目录结构 & 文件说明

| 路径                          | 作用        |
| --------------------------- | --------- |
| `/etc/nginx/nginx.conf`     | 主配置文件     |
| `/etc/nginx/conf.d/*.conf`  | 虚拟主机配置    |
| `/var/www/html/`            | Web 站点根目录 |
| `/var/log/nginx/access.log` | 访问日志      |
| `/var/log/nginx/error.log`  | 错误日志      |

必要命令：

```bash
nginx -t          # 检查配置语法
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
```

---

# 三、核心功能与操作

## 3.1 核心功能 A —— 反向代理与负载均衡

### ① 反向代理示例

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### ② 负载均衡示例（轮询）

```nginx
upstream backend {
    server 192.168.1.2;
    server 192.168.1.3;
    server 192.168.1.4;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

支持多种算法：

```
默认 round-robin（轮询）
ip_hash （会话黏性）
least_conn （最少连接）
hash $uri consistent;
```

---

## 3.2 功能 B —— HTTPS、限流、缓存、安全策略等

### 开启 HTTPS（Let’s Encrypt）

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d www.example.com
```

### 限流 & 防刷

```nginx
limit_req_zone $binary_remote_addr zone=req_limit:10m rate=5r/s;

server {
    location /api/ {
        limit_req zone=req_limit burst=10 nodelay;
    }
}
```

### 缓存加速

```nginx
proxy_cache_path /data/nginx/cache keys_zone=mycache:10m max_size=1g;

location / {
    proxy_cache mycache;
    proxy_pass http://backend;
}
```

---

# 四、进阶应用与案例

## 4.1 企业级工作流：以微服务 API 网关为例

| 阶段 | 操作内容           | 工具                     |
| -- | -------------- | ---------------------- |
| 1  | DNS 与 HTTPS 配置 | Nginx + Certbot        |
| 2  | 微服务代理与路由       | upstream + proxy_pass  |
| 3  | 服务健康检查         | max_fails fail_timeout |
| 4  | 灰度发布           | weight、hash            |
| 5  | 性能优化           | 缓存、keepalive           |
| 6  | 安全防护           | TLS、限流、WAF             |

架构图（简化）：

```
Client -> Nginx -> Load Balancer -> Microservices -> DB/Redis
```

---

## 4.2 FAQ 问题排查与性能优化建议

| 问题                  | 原因          | 解决办法                   |
| ------------------- | ----------- | ---------------------- |
| 502 Bad Gateway     | 上游服务挂掉或接口错误 | 检查后端                   |
| 504 Gateway Timeout | 响应超时        | proxy_read_timeout 修改  |
| HTTPS 访问慢           | TLS 配置不当    | HTTP/2 + session cache |
| CPU 过高              | 日志或正则耗时     | 减少正则、使用 map            |

性能优化重点：

```
worker_processes auto;
worker_connections 65535;
use epoll;
gzip on;
keepalive_timeout 65;
tcp_nopush on;
```

---

# 五、总结与资源

## 5.1 总结 & 后续学习路线

如果把业务系统看作高速公路，那么 Nginx 就是交通指挥中心：**转发、调度、限速、收费、安全、缓冲区。**

学习路线建议：

```
入门：安装 + 静态站点
进阶：反向代理 + HTTPS
高阶：缓存 + 限流 + 高可用
专家：OpenResty + Lua + 网关扩展
```

推荐学习资源（官方）：

* 官方文档：[https://nginx.org/en/docs/](https://nginx.org/en/docs/)
* 配置示例仓库： [https://github.com/nginxinc](https://github.com/nginxinc)
