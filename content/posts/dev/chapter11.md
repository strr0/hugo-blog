---
title: "基于 Sa-Token 实现单点登录"
date: 2024-06-16T14:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 1 介绍

### 1.1 Sa-Token

Sa-Token 是一个轻量级 Java 权限认证框架，主要解决：登录认证、权限认证、单点登录、OAuth2.0、分布式Session会话、微服务网关鉴权等一系列权限相关问题。

### 1.2 Sa-Token SSO

单点登录就是一套机制将N个系统的认证授权互通共享，让用户在一个系统登录之后，便可以畅通无阻的访问其它所有系统。Sa-Token 单点登录的工作原理：用户进入 A 系统，若未登录则将当前路径作为 redirect 参数跳转到授权中心，授权中心登录成功之后会颁发一个 ticket 重定向到 A 系统，A 系统拿到 ticket 并在授权中心校验通过之后可以得到授权中心登录用户 id，即说明用户在授权中心登录成功，然后可以在 A 系统上执行登录成功的后续逻辑。

## 2 示例

### 2.1 SSO 统一认证中心（服务端）

新建 sso-server 模块

引入依赖
```
<!-- Sa-Token 权限认证, 在线文档：https://sa-token.cc/ -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot3-starter</artifactId>
    <version>1.38.0</version>
</dependency>

<!-- Sa-Token 插件：整合SSO -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-sso</artifactId>
    <version>1.38.0</version>
</dependency>
```
application 配置
```
# 端口
server:
    port: 9000
# Sa-Token 配置
sa-token: 
    sso-server:
        # Ticket有效期 (单位: 秒)，默认五分钟
        ticket-timeout: 300
        # 所有允许的授权回调地址
        allow-url: "*"
        # 是否打开模式三
        is-http: true
    sign:
        # API 接口调用秘钥
        secret-key: kQwIOrYvnXmSDkwEiFngrKidMcdrgKor
```
新建 SsoServerController
```
/**
 * Sa-Token-SSO Server端 Controller 
 */
@RestController
public class SsoServerController {
    private final RestTemplate restTemplate = new RestTemplate();

    /**
     * SSO-Server端：处理所有SSO相关请求 (下面的章节我们会详细列出开放的接口) 
     */
    @RequestMapping("/sso/*")
    public Object ssoRequest() {
        return SaSsoServerProcessor.instance.dister();
    }
    
    /**
     * 配置SSO相关参数 
     */
    @Autowired
    private void configSso(SaSsoServerConfig ssoServer) {
        // 配置：未登录时返回的View 
        ssoServer.notLoginView = () -> {
            String msg = "当前会话在SSO-Server端尚未登录，请先访问"
                    + "<a href='/sso/doLogin?name=sa&pwd=123456' target='_blank'> doLogin登录 </a>"
                    + "进行登录之后，刷新页面开始授权";
            return msg;
        };
        
        // 配置：登录处理函数 
        ssoServer.doLoginHandle = (name, pwd) -> {
            // 此处仅做模拟登录，真实环境应该查询数据进行登录 
            if("sa".equals(name) && "123456".equals(pwd)) {
                StpUtil.login(10001);
                return SaResult.ok("登录成功！").setData(StpUtil.getTokenValue());
            }
            return SaResult.error("登录失败！");
        };
        
        // 配置 Http 请求处理器 （在模式三的单点注销功能下用到，如不需要可以注释掉） 
        ssoServer.sendHttp = url -> {
            try {
                System.out.println("------ 发起请求：" + url);
                String resStr = restTemplate.getForObject(url, String.class);
                System.out.println("------ 请求结果：" + resStr);
                return resStr;
            } catch (Exception e) {
                e.printStackTrace();
                return null;
            }
        };
    }
    
}
```

### 2.2 SSO 客户端

新建 sso-client 模块（或使用已有的模块）

引入依赖
```
<!-- Sa-Token 权限认证, 在线文档：https://sa-token.cc/ -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot3-starter</artifactId>
    <version>1.38.0</version>
</dependency>

<!-- Sa-Token 插件：整合SSO -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-sso</artifactId>
    <version>1.38.0</version>
</dependency>
```
application 配置
```
# 端口
server:
    port: 9001
# sa-token配置 
sa-token:
    # SSO-相关配置
    sso-client:
        # SSO-Server端 统一认证地址
        auth-url: http://localhost:9000/sso/auth
        # 使用 Http 请求校验ticket (模式三)
        is-http: true
        # SSO-Server端 ticket校验地址
        check-ticket-url: http://localhost:9000/sso/checkTicket
        # 单点注销地址
        slo-url: http://localhost:9000/sso/signout
        # 查询数据地址
        get-data-url: http://localhost:9000/sso/getData
    sign:
        # API 接口调用秘钥
        secret-key: kQwIOrYvnXmSDkwEiFngrKidMcdrgKor
```
新建 SsoClientController
```
/**
 * Sa-Token-SSO Client端 Controller 
 */
@RestController
public class SsoClientController {
    private final RestTemplate restTemplate = new RestTemplate();

    // 首页
    @RequestMapping("/")
    public String index() {
        String str = "<h2>Sa-Token SSO-Client 应用端</h2>" +
                "<p>当前会话是否登录：" + StpUtil.isLogin() + "</p>" +
                "<p><a href=\"javascript:location.href='/sso/login?back=' + encodeURIComponent(location.href);\">登录</a> " +
                "<a href='/sso/logout?back=self'>注销</a></p>";
        return str;
    }

    /*
     * SSO-Client端：处理所有SSO相关请求
     * 		http://{host}:{port}/sso/login			-- Client端登录地址，接受参数：back=登录后的跳转地址
     * 		http://{host}:{port}/sso/logout			-- Client端单点注销地址（isSlo=true时打开），接受参数：back=注销后的跳转地址
     * 		http://{host}:{port}/sso/logoutCall		-- Client端单点注销回调地址（isSlo=true时打开），此接口为框架回调，开发者无需关心
     */
    @RequestMapping("/sso/*")
    public Object ssoRequest() {
        return SaSsoClientProcessor.instance.dister();
    }

    // 配置SSO相关参数
    @Autowired
    private void configSso(SaSsoClientConfig ssoClient) {
        // 配置Http请求处理器
        ssoClient.sendHttp = url -> {
            System.out.println("------ 发起请求：" + url);
            String resStr = restTemplate.getForObject(url, String.class);
            System.out.println("------ 请求结果：" + resStr);
            return resStr;
        };
    }
}
```

### 2.3 测试及客户端自定义配置

完成上述配置后，分别启动 sso-server 和 sso-client，然后访问 http://localhost:9001 进入客户端首页，点击登录即自动跳转到授权中心，授权中心登录成功后带 ticket 重定向回 /sso/login，此路径为客户端默认登录地址，在没有额外配置的情况下，他会默认对 ticket 进行校验并在校验成功后在客户端登录。  
如果需要对客户端登录做自定义的逻辑操作，首先需要修改跳转到授权中心的 redirect 参数（即重定向地址），然后配置自定义 ticket 处理，如  
```
    // 根据ticket进行登录
    @RequestMapping("/sso/doLoginByTicket")
    public SaResult doLoginByTicket(String ticket) {
        SaCheckTicketResult ctr = SaSsoClientProcessor.instance.checkTicket(ticket, "/sso/doLoginByTicket");
        // 自定义登录逻辑
        StpUtil.login(ctr.loginId, ctr.remainSessionTimeout);
        return SaResult.data(StpUtil.getTokenValue());
    }
```