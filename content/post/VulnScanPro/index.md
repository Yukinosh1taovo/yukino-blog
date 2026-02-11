+++
date = '2026-01-04T22:42:24+08:00'
draft = false
title = '毕业设计-网络漏洞扫描与渗透测试平台'

+++

# 毕业设计-网络漏洞扫描与渗透测试平台

该平台的核心目标是将分散的安全工具和手动流程，集成到一个**自动化、流程化、可视化**的统一系统中。其主要功能模块可分为以下几个方面：

**1. 用户与权限管理**
这是系统安全与协作的基础。平台提供基于角色的访问控制，通常包括：

- **管理员**：拥有最高权限，可以管理所有用户、查看所有任务和报告、配置系统参数。
- **安全工程师/普通用户**：可以创建和管理自己的扫描任务，查看自己任务的报告。

**2. 目标管理与扫描任务调度**
这是平台的核心驱动模块，负责将用户意图转化为可执行的动作。

- **任务创建**：用户通过Web界面指定扫描目标（支持单IP、域名、IP段或URL列表）。
- **策略选择**：提供预设扫描策略，如“快速扫描”（仅端口）、“Web深度扫描”、“全面扫描”（端口+漏洞）等。
- **任务调度与队列**：支持即时、定时或周期性的扫描任务。系统会管理一个任务队列，处理并发扫描请求。

**3. 自动化资产发现与信息收集**
平台自动执行侦察阶段的工作，替代人工使用多个工具。

- **端口与服务发现**：集成类似Nmap的引擎，自动探测目标开放的端口、识别运行的服务及其版本。
- **Web应用识别**：对发现的Web服务（如80， 443， 8080端口），进行路径爬取、技术栈（框架、中间件、前端库）指纹识别。
- **资产关联与入库**：将发现的所有主机、域名、服务等信息，结构化地存入数据库，形成清晰的资产清单。

**4. 智能化漏洞检测**
这是平台的技术核心，集成了多种检测手段。

- **已知漏洞扫描**：基于插件或规则库，对识别的服务版本进行匹配，查找已知的公开漏洞。
- **Web应用漏洞扫描**：自动化检测SQL注入、跨站脚本、命令注入、不安全配置等OWASP Top 10风险。
- **弱口令检测**：可对常见的服务（如SSH， FTP， MySQL， Redis）进行字典爆破或弱口令检查。
- **结果去重与验证**：对扫描出的漏洞进行初步的自动验证（如通过无害的Payload触发），并合并重复发现，降低误报。

**5. 数据集中化展示与报告生成**
提供统一的可视化界面，告别分散的命令行输出和日志文件。

- **实时监控仪表盘**：展示平台整体状态，如任务执行情况、资产总数、漏洞风险分布（饼图）、近期活动等。
- **资产与漏洞管理台**：以列表和详情页的形式，清晰展示所有发现的资产和漏洞。漏洞可按风险等级、类型、资产等维度筛选和排序。
- **专业报告生成**：一键生成结构化的渗透测试报告，支持HTML或PDF格式。报告通常包括概述、测试方法、发现漏洞的详细描述、风险评级、证据截图（如有）及修复建议。



## 一、springboot项目创建 (环境搭建)

[springboot中文文档](https://www.spring-doc.cn/projects/spring-boot#overview)

[spring官方文档](https://spring.io/guides)

创建springboot项目 

选择springboot 3.0.2 + jdk17

![image-20260104105925142](image-20260104105925142.webp)

添加如下依赖

![image-20260104105942083](image-20260104105942083.webp)

连接好数据库后创建数据库 数据库名为 vulnscan_platform

``` sql
create schema vulnscan_platform
```

设计好数据库并创建后 创建好标准springboot项目目录结构

![image-20260124142032499](C:\Files\repos\yukino-blog\content\post\VulnScanPro\image-20260124142032499.webp)



## 二、 数据库分析创建

根据功能分析预估 设计以下数据库表结构 

![](C:\Files\repos\yukino-blog\content\post\VulnScanPro\image-20260124142637438.webp)

以下为对每张表的详细设计

### 1. 用户表

|  **字段名**   |   **类型**   | 键值 | 空/非空  | **默认值** |             描述             |
| :-----------: | :----------: | :--: | :------: | :--------: | :--------------------------: |
|      id       |    bigint    | 主键 | NOT NULL |            |         用户唯一标识         |
|   username    | varchar(50)  |      | NOT NULL |            |       用户名，用于登录       |
| password_hash | varchar(255) |      | NOT NULL |            |         加密后的密码         |
|     email     | varchar(100) |      |          |    NULL    |           电子邮箱           |
|     role      | varchar(20)  |      | NOT NULL |            |      角色：ADMIN, USER       |
|   is_active   |  tinyint(1)  |      | NOT NULL |            | 账户是否启用（1启用，0禁用） |
|  created_at   |  timestamp   |      | NOT NULL |            |         账户创建时间         |
|  updated_at   |  timestamp   |      |          |    NULL    |         最后更新时间         |



```sql
-- 1. 用户表
CREATE TABLE `users` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '用户唯一标识',
  `username` varchar(50) NOT NULL COMMENT '用户名，用于登录',
  `password_hash` varchar(255) NOT NULL COMMENT '加密后的密码',
  `email` varchar(100) DEFAULT NULL COMMENT '电子邮箱',
  `role` varchar(20) NOT NULL DEFAULT 'USER' COMMENT '角色：ADMIN, USER',
  `is_active` tinyint(1) NOT NULL DEFAULT '1' COMMENT '账户是否启用（1启用，0禁用）',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '账户创建时间',
  `updated_at` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_users_username` (`username`),
  KEY `idx_users_role` (`role`),
  KEY `idx_users_created_at` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='系统用户表';
```

### 2. 扫描任务主表



```mysql
-- 2. 扫描任务主表
CREATE TABLE `scan_tasks` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '任务唯一标识',
  `user_id` bigint NOT NULL COMMENT '任务创建者ID',
  `task_name` varchar(100) NOT NULL COMMENT '任务名称',
  `target` text NOT NULL COMMENT '扫描目标（IP、域名、URL，支持逗号分隔）',
  `scan_policy` varchar(50) NOT NULL DEFAULT 'FULL' COMMENT '扫描策略：QUICK, FULL, WEB_ONLY等',
  `status` varchar(20) NOT NULL DEFAULT 'PENDING' COMMENT '任务状态：PENDING, RUNNING, COMPLETED, FAILED, CANCELLED',
  `progress` tinyint DEFAULT '0' COMMENT '任务进度（0-100）',
  `start_time` timestamp NULL DEFAULT NULL COMMENT '任务开始执行时间',
  `end_time` timestamp NULL DEFAULT NULL COMMENT '任务结束时间',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '任务创建时间',
  PRIMARY KEY (`id`),
  KEY `idx_tasks_user_id` (`user_id`),
  KEY `idx_tasks_status` (`status`),
  KEY `idx_tasks_created_at` (`created_at`),
  CONSTRAINT `fk_tasks_user_id` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='扫描任务主表';
```

### 3. 任务状态日志表（用于详细追踪）

```sql
CREATE TABLE `task_status_logs` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '日志ID',
  `task_id` bigint NOT NULL COMMENT '关联的任务ID',
  `stage` varchar(50) NOT NULL COMMENT '当前阶段：PORT_SCAN, WEB_CRAWL, VULN_SCAN等',
  `status` varchar(20) NOT NULL COMMENT '阶段状态：START, SUCCESS, WARNING, ERROR',
  `message` text COMMENT '状态详细信息或错误日志',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '日志记录时间',
  PRIMARY KEY (`id`),
  KEY `idx_logs_task_id` (`task_id`),
  KEY `idx_logs_created_at` (`created_at`),
  CONSTRAINT `fk_logs_task_id` FOREIGN KEY (`task_id`) REFERENCES `scan_tasks` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='任务状态详细日志表';
```

### 4. 资产表（发现的主机）

```sql
CREATE TABLE `assets` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '资产ID',
  `task_id` bigint NOT NULL COMMENT '所属扫描任务ID',
  `ip_address` varchar(45) DEFAULT NULL COMMENT 'IP地址（支持IPv6）',
  `hostname` varchar(255) DEFAULT NULL COMMENT '主机名',
  `os_guess` varchar(100) DEFAULT NULL COMMENT '猜测的操作系统',
  `last_seen` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '最后一次被发现的时间',
  PRIMARY KEY (`id`),
  KEY `idx_assets_task_id` (`task_id`),
  KEY `idx_assets_ip` (`ip_address`),
  CONSTRAINT `fk_assets_task_id` FOREIGN KEY (`task_id`) REFERENCES `scan_tasks` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='发现的网络资产表';
```

### 5. 端口表

```sql
-- 5. 端口表
CREATE TABLE `ports` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '端口记录ID',
  `asset_id` bigint NOT NULL COMMENT '所属资产ID',
  `port_number` int NOT NULL COMMENT '端口号',
  `protocol` varchar(10) DEFAULT 'tcp' COMMENT '协议：tcp, udp',
  `service_name` varchar(100) DEFAULT NULL COMMENT '服务名称（如ssh, http）',
  `version_info` varchar(255) DEFAULT NULL COMMENT '服务版本信息',
  `state` varchar(20) NOT NULL DEFAULT 'unknown' COMMENT '端口状态：open, closed, filtered, unknown',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_ports_asset_port` (`asset_id`, `port_number`, `protocol`), -- 防止重复记录
  KEY `idx_ports_service` (`service_name`),
  CONSTRAINT `fk_ports_asset_id` FOREIGN KEY (`asset_id`) REFERENCES `assets` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='资产开放端口及服务表';
```

### 6. 漏洞表

```sql
CREATE TABLE `vulnerabilities` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '漏洞记录ID',
  `task_id` bigint NOT NULL COMMENT '发现该漏洞的任务ID',
  `asset_id` bigint DEFAULT NULL COMMENT '关联的资产ID（可为空，如纯域名漏洞）',
  `port_id` bigint DEFAULT NULL COMMENT '关联的端口ID（可为空）',
  `vuln_type` varchar(50) NOT NULL COMMENT '漏洞类型：SQLi, XSS, WEAK_PASSWORD等',
  `title` varchar(255) NOT NULL COMMENT '漏洞标题',
  `risk_level` varchar(20) NOT NULL DEFAULT 'MEDIUM' COMMENT '风险等级：CRITICAL, HIGH, MEDIUM, LOW, INFO',
  `is_fixed` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否已修复（0未修复，1已修复）',
  `discovered_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '漏洞发现时间',
  PRIMARY KEY (`id`),
  KEY `idx_vulns_task_id` (`task_id`),
  KEY `idx_vulns_asset_id` (`asset_id`),
  KEY `idx_vulns_risk_level` (`risk_level`), -- 便于按风险级别筛选
  KEY `idx_vulns_discovered_at` (`discovered_at`),
  CONSTRAINT `fk_vulns_task_id` FOREIGN KEY (`task_id`) REFERENCES `scan_tasks` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_vulns_asset_id` FOREIGN KEY (`asset_id`) REFERENCES `assets` (`id`) ON DELETE SET NULL,
  CONSTRAINT `fk_vulns_port_id` FOREIGN KEY (`port_id`) REFERENCES `ports` (`id`) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='漏洞主表';
```

### 7. 漏洞详情表（与漏洞表一对一或一对多）

```sql
CREATE TABLE `vulnerability_details` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '详情ID',
  `vulnerability_id` bigint NOT NULL COMMENT '关联的漏洞ID',
  `description` text COMMENT '漏洞详细描述',
  `proof` text COMMENT '漏洞证据（如请求/响应包）',
  `recommendation` text COMMENT '修复建议',
  `cve_id` varchar(20) DEFAULT NULL COMMENT '关联的CVE编号（如CVE-2021-12345）',
  `scanner_source` varchar(50) DEFAULT NULL COMMENT '发现此漏洞的扫描器或插件名称',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_details_vuln_id` (`vulnerability_id`), -- 确保一个漏洞只有一条详情记录
  KEY `idx_details_cve_id` (`cve_id`),
  CONSTRAINT `fk_details_vuln_id` FOREIGN KEY (`vulnerability_id`) REFERENCES `vulnerabilities` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='漏洞详情表';
```

### 8. 报告表

```sql
CREATE TABLE `reports` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '报告ID',
  `task_id` bigint NOT NULL COMMENT '关联的扫描任务ID',
  `user_id` bigint NOT NULL COMMENT '报告生成者ID',
  `title` varchar(255) NOT NULL COMMENT '报告标题',
  `summary` json DEFAULT NULL COMMENT '报告摘要JSON（包含统计信息）',
  `file_path` varchar(500) DEFAULT NULL COMMENT '生成的报告文件（PDF/HTML）存储路径',
  `generated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '报告生成时间',
  PRIMARY KEY (`id`),
  KEY `idx_reports_task_id` (`task_id`),
  KEY `idx_reports_user_id` (`user_id`),
  CONSTRAINT `fk_reports_task_id` FOREIGN KEY (`task_id`) REFERENCES `scan_tasks` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_reports_user_id` FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='扫描报告表';
```

## 三、 后端功能实现

开发环境

- springboot 3.0.2
- jdk17

项目使用如下的包目录结构

```tex
com.xueqiu.vulnscanpro
├── controller       # 控制层
│   ├── api
│   └── advice
├── service          # 业务逻辑层
│   ├── impl
├── repository       # 数据访问层（对应Mapper层）
├── model            # 模型层
│   ├── entity
│   ├── dto
│   │   ├── request
│   │   └── response
│   └── vo
└── config           # 配置类等
```

总体设计

| 层                       | 类别           | 推荐名称                                                     | 功能解释                                                     |
| :----------------------- | :------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Controller** (控制层)  | 用户认证       | `AuthController`                                             | 处理用户**注册、登录、注销、令牌刷新**等认证相关的HTTP请求。 |
|                          | 扫描任务       | `ScanTaskController`                                         | 提供**扫描任务的增、删、查、改**及**启动/停止**等核心操作的API接口。 |
|                          | 资产与漏洞     | `AssetController` `VulnerabilityController`                  | 提供对**已发现的资产**和**漏洞详情**的查询、筛选、统计等只读或管理接口。 |
|                          | 报告管理       | `ReportController`                                           | 处理**扫描报告的生成、下载、列表查看和删除**等请求。         |
|                          | 系统通用       | `GlobalExceptionHandler`                                     | **全局异常处理器**，集中捕获并格式化所有未处理的异常，返回统一的错误信息格式。 |
| **Service** (业务逻辑层) | 接口（定义）   | `IUserService` `IScanTaskService` `IVulnerabilityService`    | 定义对应模块的**业务方法契约**。使用 `I` 前缀是清晰标识接口的常见做法。 |
|                          | 实现类         | `UserServiceImpl` `ScanTaskServiceImpl` `VulnerabilityServiceImpl` | **业务逻辑的核心实现**。包含具体的验证、计算、流程控制和与其他Service的交互。 |
|                          | 核心引擎服务   | `ScanEngineService`                                          | **扫描引擎的调度中心**，负责调用具体的端口扫描、Web漏洞扫描等插件或工具。 |
| **Mapper** (数据访问层)  | 数据访问接口   | `UserMapper` `ScanTaskMapper` `VulnerabilityMapper` `AssetMapper ` `ScanTaskMapper` | 用于对相应实体（表）进行**标准的增删改查操作**。             |
|                          | 自定义数据访问 | `ScanTaskCustomMapper`                                       | 当需要编写**复杂自定义查询**（如多表关联、原生SQL）时，可定义此接口及其实现。 |





### 1. entity实体类编写

#### 1.1 插件依赖导入

导入lombok、mybatis插件依赖

lombok :

mybatis:

```xml
<!--    lombok依赖    -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

<!--   mybatis依赖     -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.3.0</version>
        </dependency>
```

#### 1.2 实体类编写

依次完成对实体类的编写

```java
package com.xueqiu.vulnscanpro.model.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;

// 用户实体类
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Users {
    private Long id;
    private String username;
    private String passwordHash;
    private String email;
    private String role;
    private Boolean isActive;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

```

（其他实体类略）

### 2. 用户注册登录逻辑实现

基于 **Spring Security + JWT + MyBatis** 技术栈   实现流程图如下(*日后学习flow用代码替代图片*)

![image-20260128143553142](C:\Files\repos\yukino-blog\content\post\VulnScanPro\image-20260128143553142.webp)

![image-20260128144057877](C:\Files\repos\yukino-blog\content\post\VulnScanPro\image-20260128144057877.webp)



```flow
    A[“客户端<br>提交登录表单”] --> B[“AuthController.login()”]
    B --> C[“AuthService.login()”]
    C --> D[“1. 认证 AuthenticationManager”]
    D --> E[“CustomUserDetailsService”]
    E --> F[“UserMapper.selectByUsername()”]
    F --> G[(“数据库”)]
    G --> E
    E --> H[“Spring Security UserDetails”]
    H --> D
    D --> I[“认证成功<br>设置SecurityContext”]
    I --> J[“2. 生成令牌 JwtTokenProvider”]
    J --> K[“3. 构建响应 LoginResponse”]
    K --> B
    B --> L[“返回JSON响应给客户端”]
```





#### 2.1 配置 Spring Security 、JWT 依赖

---

***Spring Security***: 

是 Spring 生态中的**安全框架**，用于保护应用程序的安全，主要解决两个问题：

- **认证（Authentication）**：验证"你是谁"
- **授权（Authorization）**：验证"你能做什么"



***JWT(Json Web Token)***:

是一种用于**身份验证和信息传递**的开放标准（RFC 7519）。

**核心特点**：

- 自包含（包含用户信息和权限）
- 无状态（服务器不需要存储 Session）
- 跨域友好（适合前后端分离和微服务）

---

导入依赖

```xml
<!--   Spring Security依赖   -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

<!--        jwt依赖     -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.11.5</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
```



##### 2.1.1 Jwt

先在application.yml文件中设置jwt的相关参数

```yaml
jwt:
	secret: "输入64B长度的字符串作为密钥" 
	expiration-ms: 86400000 # 24小时（单位：毫秒）
```

创建JwtTokenProvider工具类 位于utils包中 用于实现Jwt的相关操作

```java
package com.xueqiu.vulnscanpro.utils;

import io.jsonwebtoken.*;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import io.jsonwebtoken.security.SecurityException;
import jakarta.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Date;
import java.util.stream.Collectors;

@Slf4j
@Component
public class JwtTokenProvider {

    @Value ("${app.jwt.secret}")
    private String jwtSecret;

    @Value("${app.jwt.expiration-ms}")
    private long jwtExpirationInMs;

    private Key key;

    @PostConstruct
    public void init(){

        byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
        log.info("原始密钥字节长度: {} 位 ({}字节)", keyBytes.length * 8, keyBytes.length);
        if (keyBytes.length < 64) { // HS512需要至少64字节
            log.error("密钥长度不足！需要至少64字节(512位)，当前仅{}字节。", keyBytes.length);
            // 方案1: 抛异常让应用启动失败
            throw new IllegalArgumentException("JWT密钥长度不足，请使用至少64字节的Base64密钥");
        }




        // 将密钥字符串转换为Key对象
        this.key = Keys.hmacShaKeyFor(jwtSecret.getBytes());

        log.info("JWT密钥初始化成功，密钥长度: {} 位", keyBytes.length * 8);
    }


    /**
     * 根据认证信息生成JWT令牌
     */
    public String generateToken(Authentication authentication) {

        UserDetails userDetails = (UserDetails) authentication.getPrincipal();

        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationInMs);

        // 将权限列表转换为逗号分隔的字符串
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));

        return Jwts.builder()
                .setSubject(userDetails.getUsername())
                .claim("authorities",authorities) // 自定义声明：权限
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .signWith(key, SignatureAlgorithm.HS512)
                .compact();
    }

    /**
     * 从 Token 中提取用户名
     */
    public String getUsernameFromToken(String token) {

        Claims claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();

        return claims.getSubject();
    }

    /**
     * 验证JWT令牌是否有效
     */
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                    .setSigningKey(key)
                    .build()
                    .parseClaimsJws(token);
            return true;
        } catch (SecurityException | MalformedJwtException e) {
            log.error("无效的JWT签名", e);
        } catch (ExpiredJwtException e) {
            log.error("JWT令牌已过期", e);
        } catch (UnsupportedJwtException e) {
            log.error("不支持的JWT令牌", e);
        } catch (IllegalArgumentException e) {
            log.error("JWT claims字符串为空", e);
        } catch (Exception e) {
            log.error("验证JWT令牌时发生错误", e);
        }
        return false;
    }

    public long getJwtExpirationInMs(){
        return jwtExpirationInMs;
    }

}

```

##### 2.1.2 Spring Security

创建SecurityConfig类 位于config包中 用于对Spring Security进行配置

```java
package com.xueqiu.vulnscanpro.config;

import com.xueqiu.vulnscanpro.filter.JwtAuthenticationFilter;
import com.xueqiu.vulnscanpro.service.impl.UserDetailsServiceImpl;
import lombok.RequiredArgsConstructor;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;


@Configuration
@EnableWebSecurity // 该注解启用 Spring Security 的 web 安全功能。
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    
    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    /**
     *
     * 配置http安全规则
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 禁用CSRF、CORS、表单登录、HTTP Basic 认证（API项目通常不需要）
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.disable())
            .formLogin(form -> form.disable())
            .httpBasic(basic -> basic.disable())

            // 配置会话管理为无状态（适合前后端分离）
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            // 配置授权规则
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/**").permitAll()  // 测试暂时允许所有接口访问

                   /*

                    // 公开访问的接口（不需要登录）
                    .requestMatchers("/api/auth/**").permitAll()  // 放行认证相关接口
                    // 其他所有请求都需要认证
                    .anyRequest().authenticated()

                    */
            )


                // 添加jwt过滤器
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)

                // ✅ 添加异常处理
                .exceptionHandling(ex -> ex
                        // 未认证时返回 401
                        .authenticationEntryPoint((request, response, authException) -> {
                            response.setStatus(401);
                            response.setContentType("application/json;charset=UTF-8");
                            response.getWriter().write("{\"error\":\"未登录，请先登录\"}");
                        })
                        // 没有权限时返回 403
                        .accessDeniedHandler((request, response, accessDeniedException) -> {
                            response.setStatus(403);
                            response.setContentType("application/json;charset=UTF-8");
                            response.getWriter().write("{\"error\":\"权限不足\"}");
                        })
                );

        return http.build();
    }


    /**
     *
     * 密码加密器配置
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        // 使用BCrypt强哈希算法加密密码
        return new BCryptPasswordEncoder();
    }


    /**
     * AuthenticationManager Bean（登录时需要用到）
     */
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }



}

```

#### 2.2 编写dto类

DTO（Data Transfer Object，数据传输对象）在你的项目中扮演着**系统各层之间安全、高效传输数据的“标准化容器”或“契约”** 的角色。它的核心价值在于**解耦**与**安全**

1. **解耦API与数据库模型（核心价值）**

你的 `User` 实体类是为了映射 `users` 数据库表而设计的，它包含 `id`、`password_hash`、`created_at`、`role` 等所有表字段。如果直接在Controller的方法参数里使用 `User` 实体来接收HTTP请求，或将 `User` 实体直接返回给前端，会导致严重问题：

- **暴露敏感信息**：前端会收到 `password_hash`、`created_at` 等不应暴露的字段。
- **绑定过于僵化**：前端传参必须严格对应实体字段，无法灵活处理如 `confirmPassword`（密码确认）这类不存在于数据库的临时字段。

**DTO的作用就是打破这种绑定**。你为注册接口专门创建 `RegisterRequest`，只包含 `username`、`password`、`email` 等注册必需字段，并添加验证注解。登录接口则使用 `LoginRequest`，只包含 `username` 和 `password`。

2. **数据验证与契约定义**

   DTO是API契约的载体。通过在DTO字段上使用注解（如 `@NotBlank`、`@Email`、`@Size`），你可以清晰声明每个接口的输入要求。

   

3. **保障数据安全与隐私**

这是DTO在安全关键项目中的重要作用。通过定义不同的DTO，你可以精确控制进出系统的数据。

- **入参安全**：防止“过度绑定”攻击。即使恶意用户多传了 `role=ADMIN` 字段，因为 `RegisterRequest` 根本没有 `role` 属性，该字段会被自动忽略。
- **出参安全**：返回给前端的应是 `UserDTO` 或 `UserProfileResponse`，而不是 `User` 实体。这样可以确保不返回 `password_hash`、`salt` 等敏感信息。

4. **优化性能与适应变化**

- **减少不必要的数据传输**：查询用户列表时，可能只需要 `id` 和 `username`，而不是用户的全部详细信息。你可以定义一个精简的 `UserSimpleDTO`，只包含这两个字段，减少网络传输量。
- **应对API演变**：当需要修改API时（例如注册需要新增“手机号”字段），你只需修改 `RegisterRequest` DTO，而无需触动 `User` 实体和数据库结构，变化被隔离在接口层。

---

导入Jakarta 数据验证依赖

```xml
<!--       Jakarta 数据验证依赖   -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
```

编写LoginRequest、RegisterRequest、LoginResponse类

```java
// LoginResponse类
package com.xueqiu.vulnscanpro.model.dto.response;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class LoginResponse {
    // JWT访问令牌
    private String accessToken;
    // 令牌类型，通常是 "Bearer"
    private String tokenType = "Bearer";
    // 令牌过期时间（毫秒时间戳）
    private Long expiresIn;
    // 用户基本信息
    private UserInfo userInfo;

    @Data
    @Builder
    public static class UserInfo{
        private Long id;
        private String username;
        private String email;
        private String role;
    }
}

```

#### 2.3 mapper层设计

使用UserMapper接口定义与用户相关的数据库操作 统一使用mybatis的xml方式实现sql

|       方法名        |                             参数                             |    返回类型    |               核心用途               |
| :-----------------: | :----------------------------------------------------------: | :------------: | :----------------------------------: |
|       insert        |                  `@Param("user") User user`                  | int (影响行数) |     **插入新用户 **需获取自增ID      |
|  selectByUsername   |             `@Param("username") String username`             |      User      |      **登录时凭用户名查询用户**      |
|     selectById      |                    `@Param("id") Long id`                    |      User      | 根据ID查询，用于服务层获取用户详情。 |
|   countByUsername   |             `@Param("username") String username`             |      int       |    **注册时检查用户名是否存在**。    |
| updateLastLoginTime | `@Param("id") Long id` `@Param("loginTime") LocalDateTime loginTime` |      int       |        **记录最后登录时间**。        |

#### 2.4 service层设计

使用AuthServiceImpl实现AuthService接口 提供用户注册登录的业务逻辑

|  方法名  |           参数            |   返回类型    |          核心用途          |
| :------: | :-----------------------: | :-----------: | :------------------------: |
| register |  RegisterRequest request  |     User      |          用户注册          |
|  login   | LoginRequest loginRequest | LoginResponse | 用户登录 返回用户认证token |



#### 2.5 controller层设计



使用AuthController类提供用户注册登录的相关接口

##### 2.5.1 register

- 基本信息

​	请求路径：/api/auth/register
​	请求方式：POST
​	接口描述：该接口用于用户登录系统，登录完毕后，系统下发JWT令牌。 

- 请求参数

  > 参数格式：json
  > 参数说明：

  |   名称   |  类型  | 是否必须 |  备注  |
  | :------: | :----: | :------: | :----: |
  | username | string |   必须   | 用户名 |
  | password | string |   必须   |  密码  |
  |  email   | string |   必须   |  邮箱  |

  请求数据样例：

  ~~~json
  {
  	"username": "yukino",
      "password": "123456",
      "email": "yukinoshitawww@gmail.com"
  }
  ~~~

- 响应数据

  参数格式：json

  参数说明：

  | 名称 | 类型   | 是否必须 | 默认值 | 备注                     | 其他信息 |
  | ---- | ------ | -------- | ------ | ------------------------ | -------- |
  | code | number | 必须     |        | 响应码, 1 成功 ; 0  失败 |          |
  | msg  | string | 非必须   |        | 提示信息                 |          |
  | data | string | 必须     |        | 用户信息                 |          |

  响应数据样例：

  ~~~json
  {
      "code": 1,
      "msg": "success",
      "data": {
          "id": 10,
          "username": "yukino",
          "passwordHash": null,
          "email": "yukinoshitawww@gmail.com",
          "role": "USER",
          "isActive": true,
          "createdAt": "2026-01-31T16:52:41.121718",
          "updatedAt": null,
          "lastLoginAt": null
      }
  }
  ~~~



##### 2.5.2 login

- 基本信息
  

​	请求路径：/api/auth/login
​	请求方式：POST
​	接口描述：该接口用于用户登录系统，登录完毕后，系统下发JWT令牌。 

- 请求参数

  > 参数格式：json
  > 参数说明：

  | 名称     | 类型   | 是否必须 | 备注   |
  | -------- | ------ | -------- | ------ |
  | username | string | 必须     | 用户名 |
  | password | string | 必须     | 密码   |

  请求数据样例：

  ~~~json
  {
  	"username": "yukino",
      "password": "123456"
  }
  ~~~

- 响应数据

  参数格式：json

  参数说明：

  | 名称 | 类型   | 是否必须 | 默认值 | 备注                             | 其他信息 |
  | ---- | ------ | -------- | ------ | -------------------------------- | -------- |
  | code | number | 必须     |        | 响应码, 1 成功 ; 0  失败         |          |
  | msg  | string | 非必须   |        | 提示信息                         |          |
  | data | string | 必须     |        | 返回的数据 , jwt令牌以及用户信息 |          |

  响应数据样例：

  ~~~json
  {
      "code": 1,
      "msg": "登录成功",
      "data": {
          "accessToken": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ5dWtpbm8iLCJhdXRob3JpdGllcyI6IlJPTEVfVVNFUiIsImlhdCI6MTc2OTg1MDE5NCwiZXhwIjoxNzY5OTM2NTk0fQ.fVXV7Ob6Of_egLBscs0uv4VbqVon1-0t4InPm4NxLmtE96TGJ_idkpQoAa9aTSdw-JAZevaLocQI_JY8gxJNfw",
          "tokenType": "Bearer",
          "expiresIn": 1769936594092,
          "userInfo": {
              "id": 10,
              "username": "yukino",
              "email": "yukinoshitawww@gmail.com",
              "role": "USER",
              "lastLoginTime": "2026-01-31T17:02:52"
          }
      }
  }
  ~~~





## 四、 前端实现

- Node.js v20.20.0
- Cursor



Node.js npm





npm create vue@latest

npm install

npm run dev



