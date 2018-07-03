---
title: java servlet으로 Front Controller 구현하기
description: Front Controller 구현하기
categories:
 - labs
tags: jsp
---

### 서블릿
```java
public class FrontController extends HttpServlet { // 서블릿 기능을 위해 HttpServlet 상속

        // HttpServlet의 service 콜백 메소드를 오버라이드
        // (get과 post 등의 모든 요청 방식은 service 메소드에 의해 호출된다.)
	@Override
	protected void service(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		// 인코딩 속성 설정
		req.setCharacterEncoding("UTF-8");
        
		// 요청 uri로부터 
                String uri = req.getRequestURI();
		uri = uri.substring(req.getContextPath().length()).split(";")[0];
		
		// 1. 커맨트 핸들러를 호출하고 그로부터 goPage를 반환 .
		// 2. 이동방식
		String goPage = null;
		ICommandHandler handler = null;
		if (uri.equals("/member/memberList.do")) {
			handler = new MemberListController();
		} else if (uri.equals("/member/memberView.do")) {
			handler = new MemberViewController();
		} else if (uri.equals("/member/memberDelete.do")) {
			handler = new MemberDeleteController();
		}
		if (handler == null) {
			resp.sendError(404, "제공하는 서비스가 아닙니다.");
			return;
		}
		goPage = handler.process(req, resp);
		if (goPage == null) {
			if (!resp.isCommitted()) {
				// resp.sendError(500,"뷰에 대한 정보가 없음.");
				throw new CommonException("뷰에 대한 정보가 없음.");
			}
		} else {
			boolean redirect = goPage.startsWith("redirect:");
			String prefix = "/WEB-INF/views/";
			String suffix =".jsp";
			if (redirect) {
				goPage = goPage.substring("redirect:".length());
				resp.sendRedirect(req.getContextPath() + goPage);
			} else {
				RequestDispatcher rd = req.getRequestDispatcher(prefix+goPage+suffix);
				rd.forward(req, resp);
			}
		}

	}
}
```

## web.xml
```xml
<web-app>
...
  <!-- Front Controller의 서블릿 등록 -->
  <servlet>
  	<servlet-name>frontController</servlet-name>
  	<servlet-class>kr.or.ddit.FrontController</servlet-class>
    <init-param>
  		<param-name>mappingInfo</param-name>
  		<param-value>kr.or.ddit.uriHandlerMapping</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup> <!-- 서버 실행 시 우선적으로 서블릿을 생성한다. -->
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>frontController</servlet-name>
    <!-- 사용자 정의 확장자구조 (.do)로 끝나는 모든 URL요청에 대해 서블릿을 실행한다. -->
  	<url-pattern>*.do</url-pattern> 
    <!--
  </servlet-mapping>
...
</web-app>
```

