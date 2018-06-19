---
title: jsp와 jdbc(2)
description: JDBC ConnectionFactory
categories:
 - labs
tags: jsp, java
---
### 개요
pooling, BasicDataSource

클라이언트의 요청으로부터 서버의 요청 분석, 데이터베이스 `접속(Connection)`과 데이터 조회,
다시 서버의 데이터 가공 과정을 거쳐 클라이언트에게 최종 응답데이터를 전송하기까지,
JDBC 프로그래밍은 많은 작업시간을 요구한다.
JDBC에서 가장 많은 네트워크 지연시간을 유발하는 것이 Connection 과정이다.
JDBC의 퍼포먼스를 향상시키기 위해 다음과 같은 방법들이 있다.

### 리펙토링
매요청마다 Connection 객체를 새로 생성하지 않고,
Connection 객체 생성을 위한 클래스(ConnectionFactory)를 분리,
 *static { 초기화 블럭 }* 에서 '한번만' 생성하고 객체 상태를 유지하기

- 장점
    - static으로 선언한 초기화 블럭은 클래스 로딩 시점에서 '한번만' 호출되기 때문에,
      반복해서 Connection 객체를 생성하지 않는다. 이 방법 만으로도 처리 속도가 꽤 향상된다.</td>
      
- 주의사항
    - Connection 객체를 생성하기 전에 Driver를 로딩하는 과정이 필요한데, Driver의 완료여부를 보장할 수 없다.
    - 한번에 시도하는 접속 요청이 많으면 익셉션이 발생한다. 
    (Driver는 부하를 막기 위해 한번에 생성할 수 있는 커넥션의 수를 제한하고 있다.)
    - connection.close() 신경쓰기
     

###  DataSource 라이브러리 이용하기
*DBCP* -  Database connection pooling services. 
아파치 `commons` 유틸의 dbcp (org.apache.commons.dbcp) - [링크](http://commons.apache.org/)

*Pooling*
Connection Pool이라는 커넥션 관리자가 연결과 해제를 직접 관리한다.
일정 수의 컨넥션 객체를 미리 생성하여 요청을 기다리다가, 요청 발생시 커넥션을 차례로 할당한다.
처리가 완료된 커넥션은 Connection Pool에 다시 반납한다.

- 장점
    - 처리속도가 비약적으로 향상된다.
    - 드라이버 로딩과 Connection 객체 생성의 과정을 한번에 해결할 수 있으므로 보다 안전하다.
    - dbcp의 BasicDataSource는 드라이버 로딩 시 사용하는 DBMS 드라이버에 대한 종속성을 낮춘다.</td>


### *코드 예시*
````java
public class ConnectionFactory {
	static String url;
	static String user;
	static String password;
	static String driverClassName;
	static BasicDataSource dataSource;

	static {
		ResourceBundle bundle = ResourceBundle
				.getBundle("kr.or.ddit.db.dbInfo");
		
		url = bundle.getString("url");
		user = bundle.getString("user");
		password = bundle.getString("password");
		driverClassName = bundle.getString("driverClassName");
		
		dataSource = new BasicDataSource();
		dataSource.setDriverClassName(driverClassName);
		dataSource.setUrl(url);
		dataSource.setUsername(user);
		dataSource.setPassword(password);
		
		int initialSize = Integer.parseInt(bundle.getString("initialSize"));
		long maxWait = Long.parseLong(bundle.getString("maxWait")); 
		int maxActive = Integer.parseInt(bundle.getString("maxActive")); 
		
        // 매니저 속성 설정
		dataSource.setInitialSize(initialSize);
		dataSource.setMaxWait(maxWait); // 풀링 사이즈를 초과하고 반납되지 않은 경우 대기시간
		dataSource.setMaxActive(maxActive); // setInitialSize한 풀링 사이즈를 초과하여 최대 수용가능한 풀
	}

	public static Connection getConnection() throws SQLException {
		// 커넥션 객체 생성
		// Simple JDBC
		// Connection conn = DriverManager.getConnection(url, user, password);
        
		// DBCP
		Connection conn = dataSource.getConnection();
		return conn;
	}
}
````
