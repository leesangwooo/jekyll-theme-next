###web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />
	<default-servlet-handler/>
	
	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
	<context:component-scan base-package="kr.or.ddit" />
	
	
	
</beans:beans>

```

###root-context.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->
	<!-- 상위 컨테이너는 하위 컨테이너에서 공유해야하는 자원들을 등록해서 관리하는 기능을 한다. -->
</beans>

```

###servlet-context.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
    <!-- 톰캣은 / 으로 uri매핑된 default servlet에서 정적인 요청을 컨트롤하는데, -->
    <!-- Spring 프레임임워크로 자동생성된 설정에 의해 모든 정적인 요청이 /resources/**으로 매핑되었다. -->
    <!-- 따라서 모든 정적인 리소스는 /resources/폴더 아래에서 관리한다. -->
	<resources mapping="/resources/**" location="/resources/" />
    <!-- 만약 Spring 프레임워크에서 읽을 수 없는 정적인 요청 (mapping 설정이 어긋난 요청)의 경우 아래 명령으로 톰캣의 기본 설정을 참조할 수 있다.  -->
    <default-servlet-handler/>
	
	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
    <!-- 어노테이션으로 등록한 bean들을 검색할 base-package -->
	<context:component-scan base-package="kr.or.ddit" />
</beans:beans>
```


###HomeController.java
```java
/**
 * Handles requests for the application home page.
 */
@Controller
public class HomeController {
	
	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);

	@RequestMapping(value = "/", method = RequestMethod.GET)
	public String home(Locale locale, Model model) {
		logger.info("Welcome home! The client locale is {}.", locale);
		
		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
		
		String formattedDate = dateFormat.format(date);
		
        // view단에서 사용할 데이터를 스코프를 선택하여 공유
		model.addAttribute("serverTime", formattedDate );
		
        // 이동할 페이지 URL 중에서 경로(/WEB-INF/views/)와 확장자명(.jsp)을 제외한 이름을 반환한다.
		return "home";
        // 경로와 확장자명은 selvlet-context.xml에서 리소스 렌더링을 위한 bean에 prefix와 suffix 프라퍼티로 등록되어 있다.
	}
}

```

```