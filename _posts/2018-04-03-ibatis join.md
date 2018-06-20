---
title: iBatis를 이용한 JOIN
description: iBatis를 이용한 JOIN
categories:
 - labs
tags: java orm
---

## iBatis를 이용한 JOIN

1.  메인데이터를 가진 테이블을 기준으로 각 `테이블의 관계`를 파악한다.
    
    - 예1) MEMBER(회원 정보 테이블)와 RPOD(상품 테이블)을 조회하여,
      <br/>특정 회원의 기본정보와 그 회원이 구매한 상품의 목록을 조회하는 경우
      <br/> 메인 데이터를 가진 테이블 : MEMBER 
      
        - 1 대 N의 구조(has many) : 회원이 구매한 상품 목록이 없을 수도(null), 많을 수도(Many) 있다.

    - 예2) 구매한 상품(PROD 테이블)의 구매자에 대한 정보(BUYER 테이블)를 조회하는 경우
        - 메인 데이터를 가진 테이블 : PROD
        
        - 1 대 1의 구조(has A) : 구매한 상품의 구매자는 한 명이다.

2. `VO 모델링` : 테이블 관계를 DomainLayer(VO)들 사이에 관계로 반영한다.
	- 1 : N (has Many) 관계의 경우 : 메인 VO에 **`List`** 타입의 변수를 선언하여 VO들을 포함 
	    - 예1.MemberVO 
	    ````java
	    List<ProdVO> prodList;
        ````
        
    - 1:1 (has A) 관계의 경우 : VO에 VO를 포함
        - 예2. ProdVO
         ````java
         BuyerVO buyer;
         ````
         
### *예시. 1 대 N의 관계*

`SQLMap.xml` 설정
- Nested Map (중첩맵) 방식

    - select 조인 쿼리 등록
    ````xml
    <select id="selectMember" resultMap="Member.memberMap" parameterClass="java.lang.String">
        SELECT
            <!--MEMBER 테이블의 칼럼 -->
            MEM_ID, MEM_PASS, MEM_NAME, MEM_REGNO1, ...
            <!--PROD 테이블의 칼럼 -->
            PROD_ID, PROD_PRICE, PROD_SALE, PROD_COST, ...
        <!--PROD의 null 값을 포함하기 위해 -->
        FROM MEMBER LEFT OUTER JOIN PROD ON(CART_PROD = PROD_ID)
        WHERE MEM_ID = #mem_id#
    </select>
    ````
    
    -  resultMap 중첩 매핑 설정 
    ````xml
    <resultMap class="memberVO" id="memberMap" groupBy="mem_id">
        <result property="mem_id" column="MEM_ID" />
        <result property="mem_pass" column="MEM_PASS" />
        <result property="mem_name" column="MEM_NAME" />
        <result property="mem_regno1" column="MEM_REGNO1" />
        ...
        <!--  memberVO와 1 대 N의 관계의 ProdVO를 매핑하여 prodList에 저장  -->
        <result property="prodList" javaType="java.util.List" resultMap="Member.prodMap"/> <!--Nested Map-->
    </resultMap>
    ````

- Nested Select (중첩 셀렉트) 방식 :
    1) 각 테이블의 SELECT 쿼리를 별도로 작성
     ````xml
      <!-- MEMBER 테이블 조회 쿼리 외, PROD 테이블과의 조인 쿼리를 추가) -->
     <select id="selectProdListByMemId" parameterClass="string" resultClass="kr.or.ddit.vo.ProdVO">
        SELECT PROD_ID, PROD_NAME, PROD_COST, PROD_SALE
        WHERE PROD_MEMBER = #mem_id# <!-- 이때 쿼리 파라미터는 아래 resultMap에서 설정 -->
     </select>
    ````
    
    2) resultMap 중첩 셀렉트 설정 
    ````xml
    <resultMap class="memberVO" id="memberMap" groupBy="mem_id">
        <result property="mem_id" column="MEM_ID" />
        <result property="mem_pass" column="MEM_PASS" />
        <result property="mem_name" column="MEM_NAME" />
        <result property="mem_regno1" column="MEM_REGNO1" />
        ...
        <!--  memberVO와 1 대 N의 관계의 ProdVO를 매핑하여 prodList에 저장  -->
        <!-- select : 위에서 등록한 조인 select 쿼리 -->
        <!-- column : select 쿼리의 파라미터가 될 칼럼 -->
        <result property="prodList" select="Member.selectProdListByMemId" column="MEM_ID"/> <!--Nested Select-->
    </resultMap>
    ````
    
    * 주의. 'N+1'의 문제. 조인 조건에 참조하는 레코드를 매번 SELECT해야하기 때문에 데이터베이스에 부하가 발생한다. 