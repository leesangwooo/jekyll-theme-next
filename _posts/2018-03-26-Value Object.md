---
title: Value Object
description: VO(Value Object)에 대한 개념 정리
categories:
 - java
tags: VO
---

##### VO(Value Object)란
계층간 데이터를 전달하는 과정에서 비슷한 성격의 값들을 한데 묶은 바구니 역할의 객체를 말한다.

엔터프라이즈 자바빈즈(Enterprise JavaBeans; EJB)의 VO 생성 규칙을 알아보자.
[EJB란?](https://ko.wikipedia.org/wiki/%EC%97%94%ED%84%B0%ED%94%84%EB%9D%BC%EC%9D%B4%EC%A6%88_%EC%9E%90%EB%B0%94%EB%B9%88%EC%A6%88) by 위키백과

##### VO 생성 규칙
1. 프로퍼티(필드, 변수) 선언
2. 접근제어자를 설정(캡슐화)
	private String propertyName;

	프로퍼티명 : 주로 카멜표기법(carmelCase)을 사용한다.
	DAO와 서비스 계층간에 사용하는 VO의 경우 DB 테이블의 칼럼명을 소문자로 변환하여 프로퍼티명으로 사용할 수 있다.
	
	또는 DBMS에서 쿼리로 해당 테이블의 모든 칼럼명을 JAVA 선언문 형태로 가져올 수 있다.
	-- 칼럼을 VO의 프로퍼티로 가져가기 위한 뷰 쿼리
		SELECT 'private ' || DECODE(DATA_TYPE, 'NUMBER', 'Integer ', 'String ')
		       || LOWER(COLUMN_NAME) || ';'
		FROM COLS
		WHERE TABLE_NAME = 'MEMBER'
		ORDER BY COLUMN_ID ASC;
	
	** 프로퍼티 타입을 선언할 시 특히 주의해야할 점
	VO의 프로퍼티 타입을 int 등의 기본형으로 선언하는 경우,
	DAO로부터 VO에 칼럼의 값을 담는 과정에서 NullPointerException이 발생할 수 있다.
	DB의 칼럼 타입이 NUMBER라고 하더라도 값이 없으면 null로 처리되어 DAO로 전달되기 때문에
	기본형 타입이 null을 처리할 수 없어서 발생하는 오류이다.
	따라서 null을 수용할 수 있는 Integer와 같은 참조형 타입의 wrapper 클래스를 사용하는 것을 권장한다. 
	
3. 프로퍼티에 대한 접근 방법을 제공 : getter, setter
	
	String getPropertyName()
	setPropertyName(String value)
	
	자동생성을 위한 이클립스 단축키(windows)
	: [alt]+[shift]+[s] - Generate Getters and Setters...

4. toString 메소드를 오버라이드
	자동생성을 위한 이클립스 단축키(windows)
	: [alt]+[shift]+[s] - Generate toString()...
	
5. 상태값을 비교하기 위해 equals메소드와 hashcode메소드 오버라이드
	자동생성을 위한 이클립스 단축키(windows)
	: [alt]+[shift]+[s] - Generate hashCode() and equals()...
	
	** 예를들어 회원정보를 조회하거나 탈퇴할 때 회원ID와 같은 매개변수와 MemberVO의 객체 상태를 
	   비교해야하는 경우가 있다. 
	   회원ID가 MemberVO의 식별자라면 불필요하게 다른 프로퍼티를 사용하지 않고 equals 메소드 정의할 수 있다. 
6. 직렬화 `implements Serializable`
	직렬화란 데이터 구조나 오브젝트 상태를 스트림 가능한 바이트 형태로 변환하는 과정이다.
	java.io.Serializable 인터페이스를 상속받은 객체는 직렬화 할 수 있는 기본 조건을 가진다.  	

