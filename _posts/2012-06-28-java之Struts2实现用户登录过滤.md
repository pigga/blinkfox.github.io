---
title: java之Struts2实现用户登录过滤
date: 2012-06-28 13:57:00 +0800 
layout: post
permalink: /blog/2012/06/28/java之Struts2实现用户登录过滤.html
categories:
  - 问题一箩筐
tags:
  - JAVA
  - Struts2
  - 登录
---

在项目中难免遇到防止用户非法登录的问题。<br/>
处理方式，在Web.xml中增加过滤器的配置
```
<!-- 用户权限过滤器配置 -->
  <filter>
        <filter-name>MyFilter</filter-name>
        <filter-class>com.bohui.ipview.action.MyFilter</filter-class>
        <init-param>
            <param-name>LOGIN_URL</param-name>
            <param-value>/jsp/login.jsp</param-value>
        </init-param>
        <init-param>
            <param-name>HOME_URL</param-name>
            <param-value>/index.jsp</param-value>
        </init-param>
        <init-param>
            <param-name>CONTROL_URL</param-name>
            <param-value></param-value>
        </init-param>
        <init-param>
        	<param-name>LOAD_URL</param-name>
            <param-value>/jsp/window.jsp</param-value>
        </init-param>
   </filter>
   <filter-mapping>
        <filter-name>MyFilter</filter-name>
        <url-pattern>*.jsp</url-pattern>
   </filter-mapping>
   <filter-mapping>
        <filter-name>MyFilter</filter-name>
        <url-pattern>*.html</url-pattern>
   </filter-mapping>
```
在具体的java类中代码如下：
```
import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

public class MyFilter implements Filter{

	private static final String LOGIN_URL = "LOGIN_URL";// 用户登录的URL地址
	private static final String HOME_URL = "HOME_URL";// 用户登录主页面的URL地址
	private static final String CONTROL_URL = "CONTROL_URL";// server控制页面
	private static final String LOAD_URL = "LOAD_URL";//下载页面
	private String logon_page;
	private String home_page;
	private String control_page;
	private String load_page;

	/* server访问的URL路径 
	 * */
	@Override
	public void destroy() {
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		HttpServletRequest req = (HttpServletRequest) request;
		HttpServletResponse resp = (HttpServletResponse) response;
		// 设置请求的参数
		req.setCharacterEncoding("UTF-8");
		resp.setContentType("text/html;charset=UTF-8");
		// 设置session的超时时间
		HttpSession session = req.getSession();
		session.setMaxInactiveInterval(60 * 60 * 24);
		PrintWriter out = resp.getWriter();
		// 得到用户请求的URL
		String request_uri = req.getRequestURI();
		// 得到web应用程序的上下文环境
		String ctxPath = req.getContextPath();
		// 去除上下文路径，得到剩余部分的路径
		String uri = request_uri.substring(ctxPath.length());
		if (uri.equals(control_page)) {
			chain.doFilter(request, response);
			return;
		}else if(uri.equals(load_page)){ 
			chain.doFilter(request, response);
			return;
		}else if (uri.equals(logon_page)) {// 判断用户访问的是否是登录界面
			// Filter只是链式处理，请求依然放行到目的地址
			chain.doFilter(request, response);
			return;
		} else {
			// 如果不是登录界面，判断用户是否已经登录
			String username = (String)session.getAttribute("username");
			String password = (String)session.getAttribute("password");
			if (null != username && null != password) {
				//System.out.println(person.getCode()+person.getPassword());
				chain.doFilter(request, response);
				return;
			} else {
				out.println("<script language=\"javaScript\">"
						+ "parent.location.href='" + ctxPath + logon_page + "'"
						+ "</script>");
				return;

			}
		}

	}

	@Override
	public void init(FilterConfig config) throws ServletException {
		// 从部署描述符中获取登录页面和首页的URI
		logon_page = config.getInitParameter(LOGIN_URL);
		home_page = config.getInitParameter(HOME_URL);
		control_page = config.getInitParameter(CONTROL_URL);
		load_page = config.getInitParameter(LOAD_URL);
		// System.out.println("logon_page:" + logon_page);
		// System.out.println("home_page:" + home_page);
		//System.out.println("control_page:" + load_page);
		if (null == logon_page || null == home_page ) {
			throw new ServletException("没有找到登录页面！");
		}
	}

}
```

这样的话，即可达到预期效果。