---
title: "Spring 整合 OAuth2"
date: 2024-12-21T23:00:00+08:00
categories: ["development"]
tags: ["java"]
draft: false
---

## 介绍

OAuth 协议为用户资源的授权提供了一个安全的、开放而又简易的标准。OAuth 的授权可以使第三方不触及到用户的账号信息（如用户名与密码），即第三方无需使用用户的用户名与密码就可以申请获得该用户资源的授权。

## 1 基础知识

### 1.1 四种角色

- Resource Owner - 资源拥有者
- Resource Server - 资源服务器
- Client - 客户端
- Authorization Server - 授权服务器

### 1.2 四种授权模式

#### 1.2.1 授权码模式（Authorization Code）

授权流程如下
```
     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)
```

(A) 客户端通过资源拥有者的用户代理将客户端标识和重定向地址发送到授权服务器

(B) 授权服务器认证资源拥有者（用户代理）和是否同意授权

(C) 获得资源拥有者同意后，授权服务器重定向用户代理到目标地址，并带上 code

(D) 客户端将 code 及重定向地址发送到授权服务器请求 access_token

(E) 授权服务器返回 access_token 到客户端

#### 1.2.2 隐式模式（Implicit）

授权流程如下
```
     +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier     +---------------+
     |         -+----(A)-- & Redirection URI --->|               |
     |  User-   |                                | Authorization |
     |  Agent  -|----(B)-- User authenticates -->|     Server    |
     |          |                                |               |
     |          |<---(C)--- Redirection URI ----<|               |
     |          |          with Access Token     +---------------+
     |          |            in Fragment
     |          |                                +---------------+
     |          |----(D)--- Redirection URI ---->|   Web-Hosted  |
     |          |          without Fragment      |     Client    |
     |          |                                |    Resource   |
     |     (F)  |<---(E)------- Script ---------<|               |
     |          |                                +---------------+
     +-|--------+
       |    |
      (A)  (G) Access Token
       |    |
       ^    v
     +---------+
     |         |
     |  Client |
     |         |
     +---------+
```

(A) 客户端通过资源拥有者的用户代理将客户端标识和重定向地址发送到授权服务器

(B) 授权服务器认证资源拥有者（用户代理）和是否同意授权

(C) 获得资源拥有者同意后，授权服务器重定向用户代理到目标地址（url 片段附带 access_token）

(D) 用户代理发送请求到资源服务器（不包含上述片段）

(E) 资源服务器返回一个带嵌入式脚本的页面（脚本能提取片段中的 access_token）

(F) 用户代理执行脚本提取 access_token

(G) 用户代理返回 access_token 到客户端

#### 1.2.3 密码模式（Resource Owner Password Credentials）

授权流程如下
```
     +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          v
          |    Resource Owner
         (A) Password Credentials
          |
          v
     +---------+                                  +---------------+
     |         |>--(B)---- Resource Owner ------->|               |
     |         |         Password Credentials     | Authorization |
     | Client  |                                  |     Server    |
     |         |<--(C)---- Access Token ---------<|               |
     |         |    (w/ Optional Refresh Token)   |               |
     +---------+                                  +---------------+
```

(A) 资源拥有者向客户端提供用户名密码

(B) 客户端将从资源拥有者得到的认证信息发送到授权服务器请求 access_token

(C) 授权服务器校验认证信息，校验通过返回 access_token

#### 1.2.4 客户端模式（Client Credentials）

授权流程如下
```
     +---------+                                  +---------------+
     |         |                                  |               |
     |         |>--(A)- Client Authentication --->| Authorization |
     | Client  |                                  |     Server    |
     |         |<--(B)---- Access Token ---------<|               |
     |         |                                  |               |
     +---------+                                  +---------------+
```

(A) 客户端将客户端的认证信息发送到授权服务器请求 access_token

(B) 授权服务器校验认证信息，校验通过返回 access_token

## 2 示例

### 2.1 OAuth2 版本

#### 2.1.1 依赖

boot 版
```xml
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.5.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-jwt</artifactId>
    <version>1.1.1.RELEASE</version>
</dependency>
```

cloud 版
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
```

#### 2.1.2 配置

Security 配置
```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user").password("password").roles("USER")
            .and()
            .withUser("admin").password("password").roles("USER", "ADMIN");
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

授权服务器配置
```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    ...

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("client")
            .secret("secret")
            .authorizedGrantTypes("password", "refresh_token")
            .scopes("read", "write");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Arrays.asList(jwtTokenEnhancer, jwtAccessTokenConverter));
        endpoints.tokenStore(tokenStore)
                .accessTokenConverter(jwtAccessTokenConverter)
                .tokenEnhancer(tokenEnhancerChain)
                .authenticationManager(authenticationManager);
    }
}
```

令牌配置
```java
@Configuration
public class JwtConfig {
    // jwt 令牌转换
    @Bean
    public JwtAccessTokenConverter getJwtAccessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("secret");
        return converter;
    }

    // jwt 令牌增强
    @Bean
    public TokenEnhancer getJwtTokenEnhancer() {
        return (token, authentication) -> {
            Map<String, Object> additionalInfo = new HashMap<>();
            additionalInfo.put("username", authentication.getPrincipal());
            ((DefaultOAuth2AccessToken) token).setAdditionalInformation(additionalInfo);
            return token;
        };
    }

    @Bean
    public TokenStore getTokenStore(JwtAccessTokenConverter converter) {
        return new JwtTokenStore(converter);
    }

    @Bean
    public DefaultTokenServices getTokenService(TokenStore tokenStore) {
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        tokenServices.setTokenStore(tokenStore);
        tokenServices.setSupportRefreshToken(true);
        return tokenServices;
    }
}
```

资源服务器配置
```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/hello").hasAnyRole("USER")
            .anyRequest().authenticated();
    }
}
```

测试资源服务器
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

### 2.2 Authorization Server 版本

#### 2.2.1 依赖

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>1.0.0</version>
</dependency>
```

#### 2.2.2 拓展密码模式

密码认证 token 信息
```java
public class OAuth2ResourceOwnerAuthenticationToken extends AbstractAuthenticationToken {
    private static final long serialVersionUID = SpringAuthorizationServerVersion.SERIAL_VERSION_UID;

    private final AuthorizationGrantType authorizationGrantType;
    private final Authentication clientPrincipal;
    private final String username;
    private final String password;
    private final Set<String> scopes;

    public OAuth2ResourceOwnerAuthenticationToken(AuthorizationGrantType authorizationGrantType, Authentication clientPrincipal,
                                                  @Nullable String username, @Nullable String password, @Nullable Set<String> scopes) {
        super(Collections.emptyList());
        Assert.notNull(authorizationGrantType, "authorizationGrantType cannot be null");
        Assert.notNull(clientPrincipal, "clientPrincipal cannot be null");
        this.authorizationGrantType = authorizationGrantType;
        this.clientPrincipal = clientPrincipal;
        this.username = username;
        this.password = password;
        this.scopes = Collections.unmodifiableSet((scopes != null) ? new HashSet<>(scopes) : Collections.emptySet());
    }

    ...
}
```

密码认证转换器
```java
public class OAuth2ResourceOwnerAuthenticationConverter implements AuthenticationConverter {
    @Override
    public Authentication convert(HttpServletRequest request) {
        // grant_type (REQUIRED)
        String grantType = request.getParameter(OAuth2ParameterNames.GRANT_TYPE);
        if (!AuthorizationGrantType.PASSWORD.getValue().equals(grantType)) {
            return null;
        }

        MultiValueMap<String, String> parameters = OAuth2EndpointUtils.getParameters(request);

        // username (REQUIRED)
        String username = parameters.getFirst(OAuth2ParameterNames.USERNAME);
        if (!StringUtils.hasText(username) || parameters.get(OAuth2ParameterNames.USERNAME).size() != 1) {
            OAuth2EndpointUtils.throwError(OAuth2ErrorCodes.INVALID_REQUEST, OAuth2ParameterNames.USERNAME, OAuth2EndpointUtils.ACCESS_TOKEN_REQUEST_ERROR_URI);
        }

        // password (REQUIRED)
        String password = parameters.getFirst(OAuth2ParameterNames.PASSWORD);
        if (!StringUtils.hasText(password) || parameters.get(OAuth2ParameterNames.PASSWORD).size() != 1) {
            OAuth2EndpointUtils.throwError(OAuth2ErrorCodes.INVALID_REQUEST, OAuth2ParameterNames.PASSWORD, OAuth2EndpointUtils.ACCESS_TOKEN_REQUEST_ERROR_URI);
        }

        // scope (OPTIONAL)
        String scope = parameters.getFirst(OAuth2ParameterNames.SCOPE);
        if (StringUtils.hasText(scope) && parameters.get(OAuth2ParameterNames.SCOPE).size() != 1) {
            OAuth2EndpointUtils.throwError(OAuth2ErrorCodes.INVALID_REQUEST, OAuth2ParameterNames.SCOPE,
                    OAuth2EndpointUtils.ACCESS_TOKEN_REQUEST_ERROR_URI);
        }
        Set<String> scopes = null;
        if (StringUtils.hasText(scope)) {
            scopes = new HashSet<>(Arrays.asList(StringUtils.delimitedListToStringArray(scope, " ")));
        }

        // 用于获取 client
        Authentication clientPrincipal = SecurityContextHolder.getContext().getAuthentication();
        if (clientPrincipal == null) {
            OAuth2EndpointUtils.throwError(OAuth2ErrorCodes.INVALID_REQUEST, OAuth2ErrorCodes.INVALID_CLIENT,
                    OAuth2EndpointUtils.ACCESS_TOKEN_REQUEST_ERROR_URI);
        }

        return new OAuth2ResourceOwnerAuthenticationToken(AuthorizationGrantType.PASSWORD, clientPrincipal, username, password, scopes);
    }
}
```

密码认证处理
```java
public class OAuth2ResourceOwnerAuthenticationProvider implements AuthenticationProvider {
    private static final Log LOGGER = LogFactory.getLog(OAuth2ResourceOwnerAuthenticationProvider.class);
    private static final String ERROR_URI = "https://datatracker.ietf.org/doc/html/rfc6749#section-5.2";
    private static final OAuth2TokenType ID_TOKEN_TOKEN_TYPE = new OAuth2TokenType(OidcParameterNames.ID_TOKEN);

    private final AuthenticationManager authenticationManager;
    private final OAuth2AuthorizationService authorizationService;
    private final OAuth2TokenGenerator<? extends OAuth2Token> tokenGenerator;

    public OAuth2ResourceOwnerAuthenticationProvider(AuthenticationManager authenticationManager, OAuth2AuthorizationService authorizationService,
                                                     OAuth2TokenGenerator<? extends OAuth2Token> tokenGenerator) {
        Assert.notNull(authorizationService, "authorizationService cannot be null");
        Assert.notNull(tokenGenerator, "tokenGenerator cannot be null");
        this.authenticationManager = authenticationManager;
        this.authorizationService = authorizationService;
        this.tokenGenerator = tokenGenerator;
    }

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        OAuth2ResourceOwnerAuthenticationToken resourceOwnerAuthentication = (OAuth2ResourceOwnerAuthenticationToken) authentication;

        OAuth2ClientAuthenticationToken clientPrincipal = OAuth2AuthenticationProviderUtils
                .getAuthenticatedClientElseThrowInvalidClient(resourceOwnerAuthentication);
        RegisteredClient registeredClient = clientPrincipal.getRegisteredClient();

        if (LOGGER.isTraceEnabled()) {
            LOGGER.trace("Retrieved registered client");
        }

        if (!registeredClient.getAuthorizationGrantTypes().contains(AuthorizationGrantType.PASSWORD)) {
            throw new OAuth2AuthenticationException(OAuth2ErrorCodes.UNAUTHORIZED_CLIENT);
        }

        Authentication usernamePasswordAuthentication = null;
        try {
            usernamePasswordAuthentication = authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(resourceOwnerAuthentication.getUsername(), resourceOwnerAuthentication.getPassword()));
        } catch (AuthenticationException e) {
            throw new OAuth2AuthenticationException(OAuth2ErrorCodes.ACCESS_DENIED);
        }

        Set<String> authorizedScopes = registeredClient.getScopes();
        Set<String> requestedScopes = resourceOwnerAuthentication.getScopes();
        if (!CollectionUtils.isEmpty(requestedScopes)) {
            Set<String> unauthorizedScopes = requestedScopes.stream().filter(scope -> !authorizedScopes.contains(scope)).collect(Collectors.toSet());
            if (!CollectionUtils.isEmpty(unauthorizedScopes)) {
                throw new OAuth2AuthenticationException(OAuth2ErrorCodes.INVALID_SCOPE);
            }
        }

        if (LOGGER.isTraceEnabled()) {
            LOGGER.trace("Validated token request parameters");
        }

        // @formatter:off
        DefaultOAuth2TokenContext.Builder tokenContextBuilder = DefaultOAuth2TokenContext.builder()
                .registeredClient(registeredClient)
                .principal(usernamePasswordAuthentication)
                .authorizationServerContext(AuthorizationServerContextHolder.getContext())
                .authorizedScopes(requestedScopes)
                .authorizationGrantType(AuthorizationGrantType.PASSWORD)
                .authorizationGrant(resourceOwnerAuthentication);
        // @formatter:on

        // ----- Access token -----
        OAuth2TokenContext tokenContext = tokenContextBuilder.tokenType(OAuth2TokenType.ACCESS_TOKEN).build();
        OAuth2Token generatedAccessToken = this.tokenGenerator.generate(tokenContext);
        if (generatedAccessToken == null) {
            OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.SERVER_ERROR,
                    "The token generator failed to generate the access token.", ERROR_URI);
            throw new OAuth2AuthenticationException(error);
        }

        if (LOGGER.isTraceEnabled()) {
            LOGGER.trace("Generated access token");
        }

        OAuth2AccessToken accessToken = new OAuth2AccessToken(OAuth2AccessToken.TokenType.BEARER,
                generatedAccessToken.getTokenValue(), generatedAccessToken.getIssuedAt(),
                generatedAccessToken.getExpiresAt(), tokenContext.getAuthorizedScopes());

        // @formatter:off
        OAuth2Authorization.Builder authorizationBuilder = OAuth2Authorization.withRegisteredClient(registeredClient)
                .principalName(usernamePasswordAuthentication.getName())
                .authorizationGrantType(AuthorizationGrantType.PASSWORD)
                .authorizedScopes(authorizedScopes)
                .attribute(Principal.class.getName(), usernamePasswordAuthentication);
        // @formatter:on

        if (generatedAccessToken instanceof ClaimAccessor claimAccessor) {
            authorizationBuilder.token(accessToken, metadata -> metadata.put(OAuth2Authorization.Token.CLAIMS_METADATA_NAME, claimAccessor.getClaims()));
        } else {
            authorizationBuilder.accessToken(accessToken);
        }

        // ----- Refresh token -----
        OAuth2RefreshToken refreshToken = null;
        // Do not issue refresh token to public client
        if (registeredClient.getAuthorizationGrantTypes().contains(AuthorizationGrantType.REFRESH_TOKEN) &&
                !clientPrincipal.getClientAuthenticationMethod().equals(ClientAuthenticationMethod.NONE)) {
            tokenContext = tokenContextBuilder.tokenType(OAuth2TokenType.REFRESH_TOKEN).build();
            OAuth2Token generatedRefreshToken = this.tokenGenerator.generate(tokenContext);
            if (generatedRefreshToken != null) {
                if (!(generatedRefreshToken instanceof OAuth2RefreshToken)) {
                    OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.SERVER_ERROR,
                            "The token generator failed to generate a valid refresh token.", ERROR_URI);
                    throw new OAuth2AuthenticationException(error);
                }

                if (LOGGER.isTraceEnabled()) {
                    LOGGER.trace("Generated refresh token");
                }

                refreshToken = (OAuth2RefreshToken) generatedRefreshToken;
                authorizationBuilder.refreshToken(refreshToken);
            }
        }

        // ----- ID token -----
        OidcIdToken idToken;
        if (requestedScopes.contains(OidcScopes.OPENID)) {
            // @formatter:off
            tokenContext = tokenContextBuilder
                    .tokenType(ID_TOKEN_TOKEN_TYPE)
                    .authorization(authorizationBuilder.build())    // ID token customizer may need access to the access token and/or refresh token
                    .build();
            // @formatter:on
            OAuth2Token generatedIdToken = this.tokenGenerator.generate(tokenContext);
            if (!(generatedIdToken instanceof Jwt)) {
                OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.SERVER_ERROR,
                        "The token generator failed to generate the ID token.", ERROR_URI);
                throw new OAuth2AuthenticationException(error);
            }

            if (LOGGER.isTraceEnabled()) {
                LOGGER.trace("Generated id token");
            }

            idToken = new OidcIdToken(generatedIdToken.getTokenValue(), generatedIdToken.getIssuedAt(),
                    generatedIdToken.getExpiresAt(), ((Jwt) generatedIdToken).getClaims());
            authorizationBuilder.token(idToken, metadata -> metadata.put(OAuth2Authorization.Token.CLAIMS_METADATA_NAME, idToken.getClaims()));
        }
        else {
            idToken = null;
        }

        OAuth2Authorization authorization = authorizationBuilder.build();

        this.authorizationService.save(authorization);

        if (LOGGER.isTraceEnabled()) {
            LOGGER.trace("Saved authorization");
        }

        Map<String, Object> additionalParameters = Collections.emptyMap();
        if (idToken != null) {
            additionalParameters = new HashMap<>();
            additionalParameters.put(OidcParameterNames.ID_TOKEN, idToken.getTokenValue());
        }

        if (LOGGER.isTraceEnabled()) {
            LOGGER.trace("Authenticated token request");
        }

        return new OAuth2AccessTokenAuthenticationToken(registeredClient, clientPrincipal, accessToken, refreshToken, additionalParameters);
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return OAuth2ResourceOwnerAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

#### 2.2.3 配置

Security 配置
```java
@Configuration
public class WebSecurityConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder().username("user").password("password").roles("USER").build();
        UserDetails admin = User.builder().username("admin").password("password").roles("USER", "ADMIN").build();
        return new InMemoryUserDetailsManager(user, admin);
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```

授权服务器配置
```java
@Configuration
public class AuthorizationServerConfig {
    private static final Long ACCESS_TIMEOUT = 3600L;
    private static final Long REFRESH_TIMEOUT = 604800L;

    /**
     * A Spring Security filter chain for the Protocol Endpoints.
     */
    @Bean
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http, OAuth2TokenGenerator<? extends OAuth2Token> tokenGenerator,
                                                                      AuthenticationManager authenticationManager, OAuth2AuthorizationService authorizationService) throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
                .tokenEndpoint(tokenEndpoint -> {
                    tokenEndpoint.accessTokenRequestConverters(converters -> {
                        converters.add(new OAuth2AuthorizationCodeAuthenticationConverter());
                        converters.add(new OAuth2RefreshTokenAuthenticationConverter());
                        converters.add(new OAuth2ClientCredentialsAuthenticationConverter());
                        converters.add(new OAuth2ResourceOwnerAuthenticationConverter());  // 密码认证转换器
                    })
                    .authenticationProviders(providers -> {
                        providers.add(new OAuth2ResourceOwnerAuthenticationProvider(authenticationManager, authorizationService, tokenGenerator));  // 密码认证处理
                    })
                    .errorResponseHandler((request, response, exception) -> {
                        ...
                    });
                })
                .clientAuthentication(configurer -> {
                    configurer.errorResponseHandler((request, response, exception) -> {
                        ...
                    });
                })
                .oidc(Customizer.withDefaults())	// Enable OpenID Connect 1.0
                .and()
                .exceptionHandling(exceptions -> {
                    exceptions.accessDeniedHandler((request, response, exception) -> {
                        ...
                    });
                })
                .oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
        return http.build();
    }

    /**
     * An instance of RegisteredClientRepository for managing clients.
     */
    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient webClient = RegisteredClient.withId(UUID.randomUUID().toString())
                .clientId("WEB_CLIENT")
                .clientSecret("$2a$10$dnE/6hRuQHrBNQokKC5qfu/8wmhvuKAMbo8fZm5Ik7V6ZNqYPjKRi")   // WEB_SECRET
                .clientAuthenticationMethods(methods -> {
                    methods.add(ClientAuthenticationMethod.CLIENT_SECRET_BASIC);
                    methods.add(ClientAuthenticationMethod.CLIENT_SECRET_POST);
                })
                .authorizationGrantTypes(types -> {
                    types.add(AuthorizationGrantType.AUTHORIZATION_CODE);
                    types.add(AuthorizationGrantType.REFRESH_TOKEN);
                    types.add(AuthorizationGrantType.CLIENT_CREDENTIALS);
                    types.add(AuthorizationGrantType.PASSWORD);  // 密码模式
                })
                .redirectUri("http://127.0.0.1:8080/login/oauth2/code/default-client")
                .scopes(scopes -> scopes.add("web"))
                .tokenSettings(TokenSettings.builder()
                        .accessTokenTimeToLive(Duration.ofSeconds(ACCESS_TIMEOUT))
                        .refreshTokenTimeToLive(Duration.ofSeconds(REFRESH_TIMEOUT))
                        .build())
                .build();
        return new InMemoryRegisteredClientRepository(webClient);
    }

    @Bean
    public OAuth2AuthorizationService authorizationService() {
        return new InMemoryOAuth2AuthorizationService();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {
        return configuration.getAuthenticationManager();
    }

    /**
     * An instance of AuthorizationServerSettings to configure Spring Authorization Server.
     */
    @Bean
    public AuthorizationServerSettings authorizationServerSettings() {
        return AuthorizationServerSettings.builder().build();
    }

    /**
     * An instance of com.nimbusds.jose.jwk.source.JWKSource for signing access tokens.
     */
    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        RSAKey rsaKey = Jwks.generateRsa();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
    }

    /**
     * An instance of JwtDecoder for decoding signed access tokens.
     */
    @Bean
    public JwtDecoder jwtDecoder(JWKSource<SecurityContext> jwkSource) {
        return OAuth2AuthorizationServerConfiguration.jwtDecoder(jwkSource);
    }

    /**
     * 自定义jwt
     */
    @Bean
    public OAuth2TokenCustomizer<JwtEncodingContext> oAuth2TokenCustomizer() {
        return context -> {
            if (context.getPrincipal().getPrincipal() instanceof SysUserDetails user) {
                JwtClaimsSet.Builder claims = context.getClaims();
                claims.claim("userId", user.getId());
                claims.claim("username", user.getUsername());
                claims.claim("nickname", user.getNickname());
            }
        };
    }

    @Bean
    public OAuth2TokenGenerator<? extends OAuth2Token> tokenGenerator(JWKSource<SecurityContext> jwkSource, OAuth2TokenCustomizer<JwtEncodingContext> oAuth2TokenCustomizer) {
        JwtGenerator jwtGenerator = new JwtGenerator(new NimbusJwtEncoder(jwkSource));
        jwtGenerator.setJwtCustomizer(oAuth2TokenCustomizer);
        OAuth2AccessTokenGenerator accessTokenGenerator = new OAuth2AccessTokenGenerator();
        OAuth2RefreshTokenGenerator refreshTokenGenerator = new OAuth2RefreshTokenGenerator();
        return new DelegatingOAuth2TokenGenerator(jwtGenerator, accessTokenGenerator, refreshTokenGenerator);
    }
}
```

资源服务器配置
```java
@Configuration
public class ResourceServerConfig {
    @Bean
    public SecurityFilterChain resourceServerSecurityFilterChain(HttpSecurity http) throws Exception {
        // @formatter:off
        http
            .securityMatcher("/**")
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2ResourceServer -> oauth2ResourceServer
                .jwt(Customizer.withDefaults())
            );
        // @formatter:on

        return http.build();
    }
}
```

### 2.3 测试

获取 token（Authorization Server 版本的接口为 /oauth2/token）
```sh
POST http://localhost:8080/oauth/token?username=user&password=password&grant_type=password&scope=read
Accept: application/json
Authorization: Basic Y2xpZW50OnNlY3JldA==    # btoa('client:secret')
```

请求受保护的资源
```sh
GET http://localhost:8080/hello
Accept: application/json
Authorization: Bearer <access_token>
```