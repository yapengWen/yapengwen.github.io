---
title: SpringMVC 注解实现权限拦截
date: 2017-09-07 13:00:00
tags: spring
categories: spring mvc
---
最近在用SpringMvc写项目的时候，遇到一个问题，就是方法的鉴权问题，这个问题弄了一天了终于解决了，下面看下解决方法 **项目需求**：需要鉴权的地方，我只需要打个标签即可，比如只有用户登录才可以进行的操作，一般情况下我们会在执行方法时先对用户的身份进项校验，这样无形中增加了非常大的工作量，重复造轮子，有了java注解只需要在需要鉴权的方法上面打个标签即可
######  **解决方案：**
1、拦截器类：


```
package com.showshine.lake.Interceptor;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Enumeration;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import com.alibaba.druid.util.StringUtils;
import com.showshine.lake.utils.Config;
import com.showshine.lake.utils.EncriptUtil;


/** 
 * @ClassName: AdvSignCheckInterceptor 
 * @Description: TODO
 * @author: wenyapeng
 * @date: 2017年9月7日 上午10:19:00  
 */
public class AdvSignCheckInterceptor extends HandlerInterceptorAdapter{

	public final static String SIGN_KEY= "sign";
	public String success;
    public String failed;

    public boolean isMyHandler(Object handler) {
        if (!(handler instanceof HandlerMethod))
            return false;
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Interceptor interceptor = handlerMethod.getMethodAnnotation(Interceptor.class);
        if (interceptor == null)
            return false;
        if (!interceptor.name().equals(this.getClass().getSimpleName()) && !interceptor.name().equals(this.getClass().getName()))
            return false;
        success = interceptor.success();
        return true;
    }


    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (isMyHandler(handler)) {
        	String sign = request.getParameter(SIGN_KEY);
        	if(StringUtils.isEmpty(sign)){
        		return false;
        	}
        	String signkey = Config.get("adv.popularize.sign","2jmj7l5rSw0yVb/vlWAYkK/YBwk=");
        	List<String> keys = new ArrayList<String>();
        	Enumeration<String> parameterMap = request.getParameterNames();
        	
        	while(parameterMap.hasMoreElements()){
        		String param = parameterMap.nextElement();
        		if(!SIGN_KEY.equals(param)){
        			keys.add(request.getParameter(param));
        		}
        	}
        	Collections.reverse(keys);
        	StringBuffer key = new StringBuffer();
        	for(String i : keys){
        		key.append(i);
        	}
        	String creatSign = EncriptUtil.md5(key.toString()+signkey);
    		if(!creatSign.equals(sign)){
    			return false;
    		}else{
    			return true;
    		}
        }else{
        	return true;
        }
    }


    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    public static @interface Interceptor {
        public String name();

        public String success() default "";
    }

}

```


　　2、配置拦截器：需要在*-servlet.xml里面增加以下代码，如果您自定义了配置文件也可直接放到您定义的配置文件中


```
<mvc:interceptors>
	<bean class="com.showshine.lake.Interceptor.AdvSignCheckInterceptor"/>
</mvc:interceptors>
```
3、在类上面添加注解：

```
@Interceptor(name = "AdvSignCheckInterceptor")
@RequestMapping(value = "advIdfaCheck", method = RequestMethod.POST)
@ResponseBody
public JSONObject advIdfaCheck(String idfa,String advertSource,String sign){
	JSONObject retJson = new JSONObject();
	JSONObject data = advPopularizeService.idfaCheck(idfa);
	retJson.put("data", data);
	logger.info("param{idfa:"+idfa+"}" + " return_value " + retJson.toJSONString());
	return retJson;
}
```


　　注意：需要将默认的改为RequestMappingHandlerMapping，增加RequestMappingHandlerAdapter的bean 　　重新启动tomcat即可， 　　温馨提示：如果对方法需要鉴权只需要在方法上面打上@Auth，如果对类的所有方法需要鉴权，只需要在类上面打上@Auth即可。 　　那么问题来了，方法拦截器会吧静态资源一块拦截，我们需要在tomcat中进行对静态文件进行拦截如：我的解决方法是在web.xml进行配置，大家有好的方法也可以加我扣扣752432995一块探讨

