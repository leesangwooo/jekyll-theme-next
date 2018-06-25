---
title: java servlet filter(1)
description: 자바 서블릿 필터 사용하기
categories:
 - labs
tags: java
---


javax.servlet.Filter

### 서블릿 필터의 쓰임
1. Authentication Filters : 인증 작업에 사용
2. Logging and Auditing Filters : 로깅 작업에 사용
3. Image conversion Filters : 이미지 포맷, 크기 등의 변환 작업에 사용
4. Data compression Filters : 클라이언트 데이터를 DB에 넣을 때 압축하여 저장 공간을 확보하는데 사용
5. Encryption Filters : 암호화 작업에 사용
6. Tokenizing Filters : 요청을 통해 넘어온 문자열 데이터을 (일정한 의미를 가진) 토큰 단위로 쪼개는데 사용
7. Filters that trigger resource access events : 리소스 접근을 제어하는데 사용
8. XSL/T filters : 보안과 관련
9. Mime-type chain Filter

### 기본 구조
- Filter의 구현과 오버라이드 메소드
```java
// javax.servlet.Filter
public class FilterDesc implements Filter { 
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		// TODO 팔터의 초기화 생성 콜백 메서드 구현
	}
	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		// TODO 필터링 코드 작성
	}
	@Override
	public void destroy() {
		// TODO  필터 소멸 시 콜백 메서드 구현 
	}
}
```

- `web.xml`에 필터 등록하기
````xml
	<!-- 필터 체인의 첫번째 필터 -->
	<filter>
		<filter-name>필터_이름_1</filter-name>
		<filter-class>필터_퀄리파이드_네임_1</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>필터_이름_1</filter-name>
		<url-pattern>/*</url-pattern> <!-- 모든 컨텍스트의 요청에 대해 해당 필터를 거친다. -->
	</filter-mapping>
	
	<filter>
		<filter-name>secondFilter</filter-name>
		<filter-class>kr.or.ddit.filter.SecondFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>secondFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
````

- 주의
    1. 필터를 적용할 서블릿보다 먼저 태그를 생성하며, 필터 체인에 등록할 순서대로 생성한다.
    2. 클래스에 @WebFilter("/*")로 선언하여 설정을 대신할 수 있지만, 
   필터의 경우 작동 순서를 지정하여 필터체인으로 관리하는데 어노테이션(@) 방법으로는 이 순서 지정이 어렵다.

### 코드 샘플 1.  블라인드 필터
```java
/**
 * 누적 경고 횟수를 카운트하여 5회 이상인 경우 서비스 이용을 제한하는(blind) 기능을 하는 필터
 */
public class BlindFilter implements Filter {
	Map<String, Integer> blindMap; // ip, 누적 경고 횟수
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		blindMap = new LinkedHashMap<String, Integer>();
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
        // ServletRequest는 HttpServletResponse의 부모객체로서, 접근 가능한 메소드 및 변수의 종류가 제한적이므로,
        // 미리 HttpServletXXXXX로 다운캐스팅하여 사용하면 편리하다.
		HttpServletResponse resp = (HttpServletResponse)response;
		HttpServletRequest req = (HttpServletRequest)request;		
        
		String uri = req.getRequestURI();
		int len = req.getContextPath().length();
		uri = uri.substring(len);
        
		boolean pass = false;
        // blind 기능을 적용하지 않는 컨텍스트
		if (uri.equals("/") || uri.equals("/index.jsp")){
			pass = true;
			
		} else {
			// blind 기능을 적용할 컨텍스트 
			String ip = request.getRemoteAddr();
			Integer number = blindMap.get(ip); // 요청을 전달한 ip가 가진 누적 경고 횟수
			
			// blind 조건
			if(number != null && number >= 5){
				HttpSession session = req.getSession(); // 세션 스코프로 블라인드 상태를 유지한다.
				session.setAttribute("message", ip+":당신은 욕쟁이");
				// 조건에 해당하는 경우 index 페이지로 강제 이동시키기 
				resp.sendRedirect(req.getContextPath()+"/");
			} else {
				pass = true;
			}
		}
		
		if(pass){
			// blind 통과 
			chain.doFilter(request, response);
            // 필터의 특징.
            // 1. 필터는 doFilter 메소드를 통해 다음 필터 또는 목적 페이지로 이동할 수 있다.
            //    만약 doFilter 메소드가 호출되지 않으면 현재 상태에서 락이 걸려버린다!
            // 2. 필터 호출 시점을 전후로 request에 대한 필터링, response에 대한 필터링 기능이 구분된다. 
            //    request에 대한 필터링 조건 구현
            //    chain.doFilter(request, response);
            //    response에 대한 필터링 조건 구현
		}
		
	}

	@Override
	public void destroy() {
		logger.info("{} 필터 소멸", getClass().getSimpleName());

	}

}
```

### 코드샘플 2.  Character Encoding 필터
```java
public class CharacterEncodingFilter implements Filter{
	private String encoding;
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		encoding = filterConfig.getInitParameter("encoding");
        // 필터에 설정한 초기화 파라미터를 이용 인코딩 설정을 불러온다.
        // web.xml
        // <filter>
		//    <filter-name>characterEncodingFilter</filter-name>
		//    <filter-class>kr.or.ddit.filter.CharacterEncodingFilter</filter-class>
		//    <init-param>
		//       <param-name>encoding</param-name>
		//       <param-value>UTF-8</param-value>
		//    </init-param>
	    // </filter>
	    // <filter-mapping>
		//    <filter-name>characterEncodingFilter</filter-name>
		//    <url-pattern>/*</url-pattern>
	    // </filter-mapping>
		
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
        // 1. web.xml에 필터를 등록하고 uri-pattern 설정을 /*으로 하여 매핑
        // 2. 요청 데이터에 대한 인코딩 설정
		request.setCharacterEncoding(encoding);
		chain.doFilter(request, response);
		
	}

	@Override
	public void destroy() {
	}
	
}
```










