요약.[DBGuide.net](http://www.dbguide.net/db.db?cmd=view&boardUid=148198&boardConfigUid=9&categoryUid=216&boardIdx=135&boardStep=1)

## STANDARD SQL 개요

표준 SQL.

아직도 벤더별로 일부 기능의 개발이 진행 중인 경우도 있고 벤더별 특이한 기술 용어는 여전히 호환이 안 되고 있지만, ANSI/ISO SQL 표준을 통해 STANDARD JOIN을 포함한 많은 기능이 상호 벤치마킹하고 발전하면서 DBMS 간에 평준화를 이루어 가고 있다.

- 대표적인 ANSI/ISO 표준 SQL의 기능은 다음 내용을 포함한다.
	- STANDARD JOIN 기능 추가 (CROSS, OUTER JOIN 등 새로운 FROM 절 JOIN 기능들) 
	- SCALAR SUBQUERY, TOP-N QUERY 등의 새로운 SUBQUERY 기능들 
	- ROLLUP, CUBE, GROUPING SETS 등의 새로운 리포팅 기능 
	- WINDOW FUNCTION 같은 새로운 개념의 분석 기능들

## 일반 집합 연산자

![일반 집합 연산자](http://www.dbguide.net/publishing/img/knowledge/SQL_200.jpg)

### UNION
- UNION 연산은 수학적 합집합을 제공하기 위해, 공통 교집합의 중복을 없애기 위한 사전 작업으로 시스템에 부하를 주는 정렬 작업이 발생한다.
- 이후 UNION ALL 기능이 추가되었는데, 특별한 요구 사항이 없다면 공통집합을 중복해서 그대로 보여 주기 때문에
  <br/> 정렬 작업이 일어나지 않는 장점을 가진다.
  
### INTERSECTION
- 수학의 교집합으로써 두 집합의 공통집합을 추출한다.

### DIFFERENCE
- 수학의 차집합으로써 첫 번째 집합에서 두 번째 집합과의 공통집합을 제외한 부분이다.
- 대다수 벤더는 EXCEPT를, Oracle은 MINUS 용어를 사용한다.

### PRODUCT
- CROSS(ANIS/ISO 표준) PRODUCT라고 불리는 곱집합으로, JOIN 조건이 없는 경우 생길 수 있는 모든 데이터의 조합을 말한다.
- 양쪽 집합의 M*N 건의 데이터 조합이 발생

## 순수 관계 연산자

![순수 관계 연산자](http://www.dbguide.net/publishing/img/knowledge/SQL_201.jpg)

### SELECT 연산
- SQL 문장에서는 WHERE 절의 조건절 기능으로 구현이 되었다. (SELECT 연산과 SELECT 절의 의미가 다름을 유의하자.)

###  PROJECT 연산
- SQL 문장에서는 SELECT 절의 칼럼 선택 기능으로 구현이 되었다.

### JOIN 연산
- WHERE 절의 INNER JOIN 조건과 함께 FROM 절의 NATURAL JOIN, INNER JOIN, OUTER JOIN, USING 조건절, ON 조건절 등으로 가장 다양하게 발전하였다.

	- FROM 절 JOIN 형태
	````
    [예제]
    WHERE 절 JOIN 조건 SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP, DEPT WHERE EMP.DEPTNO = DEPT.DEPTNO;
    위 SQL과 아래 SQL은 같은 결과를 얻을 수 있다.
    FROM 절 JOIN 조건 SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP INNER JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO;
    INNER는 JOIN의 디폴트 옵션으로 아래 SQL문과 같이 생략 가능하다.
    SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO;
    ````
		- 과거 WHERE 절에서 JOIN 조건과 데이터 검증 조건이 같이 사용되어 용도가 불분명한 경우가 발생할 수 있었는데,
		<br/> WHERE 절의 JOIN 조건을 FROM 절의 ON 조건절로 분리하여 표시함으로써 사용자가 이해하기 쉽도록 한다.
		- NATURAL JOIN은 두 테이블 간의 동일한 이름을 갖는 모든 칼럼들에 대해 EQUI(=) JOIN을 수행한다.
		- NATURAL JOIN이 명시되면, 추가로 USING 조건절, ON 조건절, WHERE 절에서 JOIN 조건을 정의할 수 없다.
		<br/> 그리고, SQL Server에서는 지원하지 않는 기능이다.
        
    - USING 조건절
    	- NATURAL JOIN에서는 모든 일치되는 칼럼들에 대해 JOIN이 이루어지지만,
    	- FROM 절의 USING 조건절을 이용하면 같은 이름을 가진 칼럼들 중에서 원하는 칼럼에 대해서만 선택적으로 EQUI JOIN을 할 수가 있다. 			<br/>다만, 이 기능은 SQL Server에서는 지원하지 않는다.    
		````
        [예제]
        SELECT *
        FROM DEPT JOIN DEPT_TEMP
        USING (DEPTNO);
        ````
        - `USING 조건절`을 이용한 JOIN에서는 JOIN 칼럼에 대해서 `ALIAS나 테이블 명과 같은 접두사를 사용하면 SYNTAX 에러`가 발생하지만,
        - 반대로 `ON 조건절`을 사용한 JOIN의 경우는 ALIAS나 테이블 명과 같은 접두사를 사용하여 SELECT에 사용되는 칼럼을 논리적으로 명확하게 지정해주어야 한다.

	- CROSS JOIN
        - 일반 집합 연산자의 PRODUCT의 개념으로 테이블 간 JOIN 조건이 없는 경우 생길 수 있는 모든 데이터의 조합을 말한다.
        - 정상적인 데이터 모델이라면 CROSS PRODUCT가 필요한 경우는 많지 않지만, 간혹 튜닝이나 리포트를 작성하기 위해 고의적으로 사용하는 경우가 있을 수 있다.
        -  그리고 데이터웨어하우스의 개별 DIMENSION(차원)을 FACT(사실) 칼럼과 JOIN하기 전에 모든 DIMENSION의 CROSS PRODUCT를 먼저 구할 때 유용하게 사용할 수 있다.

	- OUTER JOIN
		- INNER(내부) JOIN과 대비하여 OUTER(외부) JOIN이라고 불리며, JOIN 조건에서 동일한 값이 없는 행도 반환할 때 사용할 수 있다.

### DIVIDE 연산
- 나눗셈과 비슷한 개념으로 왼쪽의 집합을 ‘XZ’로 나누었을 때, 즉 ‘XZ’를 모두 가지고 있는 ‘A’가 답이 되는 기능으로 현재 사용되지 않는다.

