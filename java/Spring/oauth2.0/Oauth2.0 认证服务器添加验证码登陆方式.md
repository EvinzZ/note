# Oauth2.0 认证服务器添加验证码登陆方式

### 发送验证码

```java
@RestController
@AllArgsConstructor
@RequestMapping
public class LoginController {

    private final RedisTemplate<String, String> redisTemplate;

    @GetMapping(value = "captcha/{phone}")
    public R captcha(@PathVariable String phone) {
        String captcha = randomCode();
        redisTemplate.opsForValue().set(phone, captcha, 600, TimeUnit.SECONDS);
        return R.ok(captcha);
    }

    private static String randomCode() {
        Random random = new Random();
        int code = random.nextInt(10000);
        DecimalFormat format = new DecimalFormat("0000");
        return format.format(code);
    }

}
```

### 登陆验证码校验过滤器 CaptchaFilter

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class CaptchaFilter extends OncePerRequestFilter {

    private final RedisTemplate<String, String> redisTemplate;
    private final UserService userService;
    private RequestMatcher requestMatcher = new AntPathRequestMatcher("/oauth/token", HttpMethod.POST.name());

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        if (requestMatcher.matches(request)) {
            String grantType = request.getParameter("grant_type");
            if (StrUtil.equalsIgnoreCase(grantType, "captcha")) {
                try {
                    verifyCaptcha(request);
                } catch (BusinessException e) {
                    log.error("登陆验证码校验异常: {}, {}", e.getCode(), e.getMsg());
                    R.failRender(e.getCode(), e.getMsg(), response, HttpStatus.INTERNAL_SERVER_ERROR.value());
                    return;
                }
            }
        }
        filterChain.doFilter(request, response);
    }

    private void verifyCaptcha(HttpServletRequest request) throws ServletRequestBindingException {
        String phone = ServletRequestUtils.getStringParameter(request, "username");
        String captcha = ServletRequestUtils.getStringParameter(request, "password");
        String cache = redisTemplate.opsForValue().get(phone);
        if (Objects.isNull(cache) || !captcha.equals(cache)) {
            throw new BusinessException("验证码校验异常");
        }
    }
}
```

### 自定义一种授权模式 CaptchaTokenGranter

- 自定义验证码授权模式
- 配置到 AuthorizationServerConfig.tokenGranter()
- 将配置好的授权列表添加到 AuthorizationServerEndpointsConfigurer 中

```java
public class CaptchaTokenGranter extends AbstractTokenGranter {

    private static final String GRANT_TYPE = "captcha";
    private UserDetailsServiceImpl userDetailsServiceImpl;

    public CaptchaTokenGranter(AuthorizationServerTokenServices tokenServices, ClientDetailsService clientDetailsService, OAuth2RequestFactory requestFactory, UserDetailsServiceImpl userDetailsServiceImpl) {
        super(tokenServices, clientDetailsService, requestFactory, GRANT_TYPE);
        this.userDetailsServiceImpl = userDetailsServiceImpl;
    }

    @Override
    protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {
        Map<String, String> requestParameters = tokenRequest.getRequestParameters();
        String username = requestParameters.getOrDefault("username", "");
        UserDetails userDetails = userDetailsServiceImpl.loadUserByUsername(username);
        if (Objects.isNull(userDetails)) {
            throw new UsernameNotFoundException("Username Not Found Exception");
        }
        // 构建用户授权信息
        Authentication user = new UsernamePasswordAuthenticationToken(userDetails.getUsername(),
                userDetails.getPassword(), userDetails.getAuthorities());
        return new OAuth2Authentication(tokenRequest.createOAuth2Request(client), user);
    }
}
```

### 将定义的授权模式添加到认证服务器核心配置 AuthorizationServerConfig 的端点配置中

AuthorizationServerConfig 其他配置已省略，详细见[Oauth2.0 认证服务器搭建](https://www.yuque.com/docs/share/30867c37-cba0-41c8-9b25-c688de629ff5)

```java
@Configuration
@EnableAuthorizationServer
@AllArgsConstructor
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
	
    ......
    
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenGranter(tokenGranter(endpoints)); //配置授权方式
    }

    /**
     * 先获取已有的五种授权列表，然后将自定义的授权方式置入
     *
     * @param endpoints AuthorizationServerEndpointsConfigurer
     * @return TokenGranter
     */
    private TokenGranter tokenGranter(final AuthorizationServerEndpointsConfigurer endpoints) {
        List<TokenGranter> granters = new ArrayList<>(Collections.singletonList(endpoints.getTokenGranter()));
        granters.add(new CaptchaTokenGranter(endpoints.getTokenServices(), endpoints.getClientDetailsService(),
                endpoints.getOAuth2RequestFactory(), userDetailsServiceImpl));
        return new CompositeTokenGranter(granters);
    }
    
    ......

}
```

