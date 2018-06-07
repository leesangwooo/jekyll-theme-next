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
	<!-- MEMBER테이블 전체 레코드를 조회하는 쿼리 -->
    <!--   조회 결과값들의 매핑 설정 : memberVO -->
    <!--   파라미터는 map 형태로 전달된다. -->
	<select id="selectMemberList" resultClass="memberVO" parameterClass="map">
    	<!-- 기본적으로 수행하는 쿼리 --> 
		SELECT MEM_ID, TO_CHAR(MEM_BIR, 'YYYY-MM-DD') MEM_BIR, MEM_NAME, MEM_ADD1, MEM_HP, MEM_MAIL, MEM_MILEAGE
		FROM MEMBER
		WHERE MEM_DELETE = 'N' 
		
        <!-- 기본 쿼리문장에 동적으로 추가되는 쿼리 -->
		<dynamic prepend=" AND " open="(" close=")"> 
        <!-- 1. WHERE 조건을 추가하는 'AND'를 앞에 붙여준다(prepend)-->
        <!-- 2. 아래 다이나믹 태그로 인해 추가되는 OR 연산자는 AND보다 연산 우선순위가 낮다. -->
		<!--   다이나믹 태그로 인해 추가되는 쿼리조각을 블럭처리 <...open="(" close=")"...>를 하지 않으면,  -->
        <!--   완성되는 쿼리는 AND 연산을 우선적으로 처리하므로, 이를 막기 위해 '(...다이나믹 쿼리...)'로 감싸준다. -->
			<isEqual property="searchType" compareValue="name">
            <!-- : 파라미터(map)의 property(key)가 'searchType'인 엔트리의 value가 -->
            <!-- name인지를 비교(compareValue)하여 같으면(isEqual) 아래 쿼리를 덧붙인다.-->
				MEM_NAME LIKE '%'||#searchWord#||'%' 
                <!-- #인라인 파라미터#는 문자열의 형태로 매핑된다. -->
                <!-- 문자를 합치는 '||'를 이용해 와일드 카드 '%'를 인라인 파라미터와 더한다.--> 
			</isEqual>
			<isEqual property="searchType" compareValue="add1">
				MEM_ADD1 LIKE '%'||#searchWord#||'%'
			</isEqual>
			<isEqual property="searchType" compareValue="all">
				MEM_NAME LIKE '%'||#searchWord#||'%' OR
				MEM_ADD1 LIKE '%'||#searchWord#||'%'
			</isEqual>
		</dynamic>
	</select>
</sqlMap>
```