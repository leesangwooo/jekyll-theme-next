##iBatis dynamic태그
dynamic태그는 파라미터의 조건에 의해 sqlMap.xml에 등록한 Query에 조건문을 추가하거나 문자열을 더하는 등 동적인 수정을 가능하게 한다.
####dynamic태그의 종류
1. 이항연산 요소
	- *`<isEqual>`* 프라퍼티와 값 또는 다른 프라퍼티가 같은지 체크.
	- *`<isNotEqual>`* 프라퍼티와 값 또는 다른 프라퍼티가 같지 않은지 체크.
	- `<isGreaterThan>` 프라퍼티가 값 또는 다른 프라퍼티보다 큰지 체크.
	- `<isGreaterEqual>` 프라퍼티가 값 또는 다른 프라퍼티보다 크거나 같은지 체크.
	- `<isLessThan>` 프라퍼티가 값 또는 다른 프라퍼티보다 작은지 체크.
	
2. 단항연산 요소
	- `<isPropertyAvailable>` 프라퍼티가 유효한지 체크(이를 테면 파라미터빈의 프라퍼티이다.)
	- `<isNotPropertyAvailable>` 프라퍼티가 유효하지 않은지 체크(이를 테면 파라미터의 프라퍼티가 아니다.)
	- *`<isNull>`* 프라퍼티가 null인지 체크
	- *`<isNotNull>`* 프라퍼티가 null이 아닌지 체크
	- `<isEmpty>` Collection, 문자열 또는 String.valueOf() 프라퍼티가 null이거나 empty(“” or size() < 1)인지 체크
	- `<isNotEmpty>` Collection, 문자열 또는 String.valueOf() 프라퍼티가 null이 아니거나 empty(“” or size() < 1)가 아닌지 체크.

3. 다른 요소
	- `<isParameterPresent>` 파라미터 객체가 존재(not null)하는지 보기위해 체크.
	- `<isNotParameterPresent>` 파라미터 객체가 존재하지(null) 않는지 보기위해 체크.
	- `<iterate>` java.util.Collection 이나 java.util.Iterator 또는 배열 구현체인 프라퍼티에 대해 반복적으로 처리

####dynamic태그의 사용
***-ex) sqlMap.xml*** (MEMBER 테이블의 레코드를 조회하기 위한 쿼리를 등록한 파일)
```xml
<sqlMap namespace="Member">
	<sql id="searchFrag">
	<dynamic prepend=" WHERE ">
		<isNotNull property="searchVO">
			<isNotEmpty property="searchVO.prod_lgu" prepend=" AND ">
				PROD_LGU = #searchVO.prod_lgu#
			</isNotEmpty>
			<isNotEmpty property="searchVO.prod_lgu" prepend=" AND ">
				PROD_BUYER = #searchVO.prod_buyer#
			</isNotEmpty>
			<isNotEmpty property="searchVO.prod_lgu" prepend=" AND ">
				PROD_BUYER LIKE '%'||#searchVO.prod_name#||'%'
			</isNotEmpty>
		</isNotNull>
	</dynamic>
	</sql>
	
	<select id="selectProdCount" resultClass="int">
		SELECT COUNT(PROD_ID)
		FROM PROD
		<include refid="searchFrag"/>
	</select>
</sqlMap>
```