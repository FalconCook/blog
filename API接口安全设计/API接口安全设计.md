## 互联网开放平台

1.需求:现在A公司与B公司进行合作，B公司需要调用A公司开放的外网接口获取数据，
如何保证外网开放接口的安全性。

2.常用解决办法：
2.1 使用加签名方式，防止篡改数据
2.2 使用Https加密传输
2.3 搭建OAuth2.0认证授权
2.4 使用令牌方式
2.5 搭建网关实现黑名单和白名单

## 使用令牌方式搭建搭建API开放平台

原理：为每个合作机构创建对应的appid、app_secret，生成对应的access_token（有效期2小时），在调用外网开放接口的时候，必须传递有效的access_token。

## 数据库表设计

```sql
CREATE TABLE `m_app` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `app_name` varchar(255) DEFAULT NULL,
  `app_id` varchar(255) DEFAULT NULL,
  `app_secret` varchar(255) DEFAULT NULL,
  `is_flag` varchar(255) DEFAULT NULL,
  `access_token` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

App_Name       表示机构名称
App_ID          应用id
App_Secret      应用密钥  （可更改） 
Is_flag           是否可用 （是否对某个机构开放）
access_token  上一次access_token

## 获取AccessToken

```java
// 创建获取getAccessToken
@RestController
@RequestMapping(value = "/auth")
public class AuthController extends BaseApiService {
	@Autowired
	private BaseRedisService baseRedisService;
	private long timeToken = 60 * 60 * 2;
	@Autowired
	private AppMapper appMapper;

	// 使用appId+appSecret 生成AccessToke
	@RequestMapping("/getAccessToken")
	public ResponseBase getAccessToken(AppEntity appEntity) {
		AppEntity appResult = appMapper.findApp(appEntity);
		if (appResult == null) {
			return setResultError("没有对应机构的认证信息");
		}
		int isFlag = appResult.getIsFlag();
		if (isFlag == 1) {
			return setResultError("您现在没有权限生成对应的AccessToken");
		}
		// ### 获取新的accessToken 之前删除之前老的accessToken
		// 从redis中删除之前的accessToken
		String accessToken = appResult.getAccessToken();
		baseRedisService.delKey(accessToken);
		// 生成的新的accessToken
		String newAccessToken = newAccessToken(appResult.getAppId());
		JSONObject jsonObject = new JSONObject();
		jsonObject.put("accessToken", newAccessToken);
		return setResultSuccessData(jsonObject);
	}

	private String newAccessToken(String appId) {
		// 使用appid+appsecret 生成对应的AccessToken 保存两个小时
		String accessToken = TokenUtils.getAccessToken();
		// 保证在同一个事物redis 事物中
		// 生成最新的token key为accessToken value 为 appid
		baseRedisService.setString(accessToken, appId, timeToken);
		// 表中保存当前accessToken
		appMapper.updateAccessToken(accessToken, appId);
		return accessToken;
	}
}
```

## 编写拦截器拦截请求,验证accessToken

```java
//验证AccessToken 是否正确
@Component
public class AccessTokenInterceptor extends BaseApiService implements HandlerInterceptor {
	@Autowired
	private BaseRedisService baseRedisService;

	/**
	 * 进入controller层之前拦截请求
	 * 
	 * @param httpServletRequest
	 * @param httpServletResponse
	 * @param o
	 * @return
	 * @throws Exception
	 */

	public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o)
			throws Exception {
		System.out.println("---------------------开始进入请求地址拦截----------------------------");
		String accessToken = httpServletRequest.getParameter("accessToken");
		// 判断accessToken是否空
		if (StringUtils.isEmpty(accessToken)) {
			// 参数Token accessToken
			resultError(" this is parameter accessToken null ", httpServletResponse);
			return false;
		}

		String appId = (String) baseRedisService.getString(accessToken);
		if (StringUtils.isEmpty(appId)) {
			// accessToken 已经失效!
			resultError(" this is  accessToken Invalid ", httpServletResponse);
			return false;
		}
		// 正常执行业务逻辑...
		return true;

	}

	public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o,
			ModelAndView modelAndView) throws Exception {
		System.out.println("--------------处理请求完成后视图渲染之前的处理操作---------------");
	}

	public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
			Object o, Exception e) throws Exception {
		System.out.println("---------------视图渲染之后的操作-------------------------0");
	}

	// 返回错误提示
	public void resultError(String errorMsg, HttpServletResponse httpServletResponse) throws IOException {
		PrintWriter printWriter = httpServletResponse.getWriter();
		printWriter.write(new JSONObject().toJSONString(setResultError(errorMsg)));
	}

}
```