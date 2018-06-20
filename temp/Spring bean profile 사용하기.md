빈의 그룹화
```xml
<bean id="idolSearchServiceImpl" lazy-init="true" class="kr.or.ddit.idol.service.IdolSearchServiceImpl">
    <constructor-arg name="dao" ref="IIdolSearchDAO" ></constructor-arg>
</bean>

<beans profile="stage">
    <bean id="IIdolSearchDAO" class="kr.or.ddit.idol.dao.IdolSearchDAO_Oracle" />
</beans>

<beans profile="dev">
    <bean id="IIdolSearchDAO" class="kr.or.ddit.idol.dao.IdolSearchDAO_MySql" />
</beans>
```
- `lazy-init` :IdolSearchServiceImpl 빈의 매개변수(<constructor-arg>의 ref 속성) 가 아직 선언되지 않은 시점에 사용되므로


```java
ConfigurableApplicationContext context = 
	new ClassPathXmlApplicationContext("kr/or/ddit/conf/firstIdolDI.xml");
context.getEnvironment().setActiveProfiles("dev");
context.refresh();

IIdolSearchService service1_1 = context.getBean(IIdolSearchService.class);
```