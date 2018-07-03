### CASE 표현

CASE 표현은 IF-THEN-ELSE 논리와 유사한 방식으로 표현식을 작성해서 SQL의 비교 연산 기능을 보완하는 역할
Oracle의 Decode 함수와 같은 기능
- `CASE SIMPLE_CASE_EXPRESSION 조건 or SEARCHED_CASE_EXPRESSION 조건 ELSE 표현절 END`
	````
    [예제] 부서 정보에서 부서 위치를 미국의 동부, 중부, 서부로 구분하라.

	SELECT LOC,
    	CASE LOC
        WHEN 'NEW YORK' THEN 'EAST'
        WHEN 'BOSTON' THEN 'EAST'
        WHEN 'CHICAGO' THEN 'CENTER'
        WHEN 'DALLAS' THEN 'CENTER'
        ELSE 'ETC'
        END as AREA
    FROM DEPT;
    ````
	````
    [예제] 사원 정보에서 급여가 2000 이상이면 보너스를 1000으로, 1000 이상이면 5000으로, 1000 미만이면 0으로 계산한다.

	SELECT ENAME, SAL,
    	CASE WHEN SAL >= 2000 THEN 1000
        ELSE (
        	CASE WHEN SAL >= 1000 THEN 500
        	ELSE 0
            END
            )
        END as BONUS
    FROM EMP;
    ````

### Null 관련 함수

#### NVL/ISNULL 함수

- 널 값을 포함하는 연산의 경우 결과 값도 널 값이다.
- Oracle의 경우 NVL 함수를 사용한다. `NVL (NULL 판단 대상,‘NULL일 때 대체값’)`
- SQL Server의 경우 ISNULL 함수를 사용한다. `ISNULL (NULL 판단 대상,‘NULL일 때 대체값’)`
    ````
    [예제] 급여와 커미션을 포함한 연봉을 계산하면서 NVL 함수의 필요성을 알아본다.
    
    SELECT ENAME 사원명, SAL 월급, COMM 커미션, (SAL * 12) + COMM 연봉A, (SAL * 12) + NVL(COMM,0) 연봉B
    FROM EMP;
    ````

### NULLIF

- 특정 값을 NULL로 대체하는 경우 : EXPR1이 EXPR2와 같으면 NULL을, 같지 않으면 EXPR1을 리턴한다.
	````
    [예제] 사원 테이블에서 MGR와 7698이 같으면 NULL을 표시하고, 같지 않으면 MGR를 표시한다.
    
    SELECT ENAME, EMPNO, MGR, NULLIF(MGR,7698) NUIF
    FROM EMP;
    ````

### COALESCE

- COALESCE 함수는 인수의 숫자가 한정되어 있지 않으며, 임의의 개수 EXPR에서 NULL이 아닌 최초의 EXPR을 나타낸다. 만일 모든 EXPR이 NULL이라면 NULL을 리턴한다.
	````
    [예제] 사원 테이블에서 커미션을 1차 선택값으로, 급여를 2차 선택값으로 선택하되 두 칼럼 모두 NULL인 경우는 NULL로 표시한다.
    
    SELECT ENAME, COMM, SAL, COALESCE(COMM, SAL) COAL 
    FROM EMP;
    ````


### ORDER BY 정렬 주의사항

- Oracle에서는 NULL 값을 가장 큰 값으로 간주하여 오름차순으로 정렬했을 경우에는 가장 마지막에,
  <br/> 내림차순으로 정렬했을 경우에는 가장 먼저 위치한다.

- 반면, SQL Server에서는 NULL 값을 가장 작은 값으로 간주하여 오름차순으로 정렬했을 경우에는 가장 먼저,
  <br/> 내림차순으로 정렬했을 경우에는 가장 마지막에 위치한다.

- 관계형 데이터베이스가 데이터를 메모리에 올릴 때 행 단위로 모든 칼럼을 가져오게 되므로,
  <br/> SELECT 절에서 일부 칼럼만 선택하더라도 ORDER BY 절에서 메모리에 올라와 있는 다른 칼럼의 데이터를 사용할 수 있다.
    - 그러나 서브쿼리의 SELECT 절에서 선택되지 않은 칼럼들은 계속 유지되는 것이 아니라
    <br/> 서브쿼리 범위를 벗어나면 더 이상 사용할 수 없게 된다. (인라인 뷰도 동일함)
    - GROUP BY 절에서 그룹핑 기준을 정의하게 되면 데이터베이스는 일반적인 SELECT 문장처럼 FROM 절에 정의된 테이블의 구조를
    <br/> 그대로 가지고 가는 것이 아니라,
    <br/> GROUP BY 절의 그룹핑 기준에 사용된 칼럼과 집계 함수에 사용될 수 있는 숫자형 데이터 칼럼들의 집합을 새로 만든다.
    <br/> GROUP BY 이후 수행 절인 SELECT 절이나 ORDER BY 절에서 개별 데이터를 사용하는 경우 에러가 발생한다.

    ````
    [예제] 한 개의 칼럼이 아닌 여러 가지 칼럼(Column)을 기준으로 정렬해본다. 먼저 키가 큰 순서대로, 키가 같은 경우 백넘버 순으로 ORDER BY 절을 적용하여 SQL 문장을 작성하는데, 키가 NULL인 데이터는 제외한다.

    SELECT PLAYER_NAME 선수이름, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키
    FROM PLAYER
    WHERE HEIGHT IS NOT NULL
    ORDER BY HEIGHT DESC, BACK_NO;
    ````

### SELECT 문장의 실행 순서

````
5. SELECT 칼럼명 [ALIAS명]
1. FROM 테이블명
2. WHERE 조건식
3. GROUP BY 칼럼(Column)이나 표현식
4. HAVING 그룹조건식
6. ORDER BY 칼럼(Column)이나 표현식;
````

SELECT 문장의 실행순서가 중요한 이유는 아래와 같다.

###  Top N 쿼리

- Oracle의 경우 정렬이 완료된 후 데이터의 일부가 출력되는 것이 아니라, 데이터의 일부가 먼저 추출된 후
  <br/> (ORDER BY 절은 결과 집합을 결정하는데 관여하지 않음) 데이터에 대한 정렬 작업이 일어나므로 주의해야 한다.
    ````
    [예제] 사원 테이블에서 급여가 높은 3명만 내림차순으로 출력하고자 하는데, 잘못 사용된 SQL의 사례이다.

    SELECT ENAME, SAL FROM EMP WHERE ROWNUM < 4 ORDER BY SAL DESC;

    [실행 결과] ENAME SAL ------ ---- ALLEN 1600 WARD 1250 SMITH 800 3개의 행이 선택되었다.
    실행 결과의 3명은 급여가 상위인 3명을 출력한 것이 아니라,
    급여 순서에 상관없이 무작위로 추출된 3명에 한해서 급여를 내림차순으로 정렬한 결과이므로
    원하는 결과를 출력한 것이 아니다.
    ````

    ````
    [예제] 정답쿼리
    SELECT ENAME, SAL
    FROM (SELECT ENAME, SAL FROM EMP ORDER BY SAL DESC)
    WHERE ROWNUM < 4 ;
    ````

- `TOP ( )`
	<br/> : 반면 SQL Server는 TOP 조건을 사용하게 되면 별도 처리 없이 관련 Order By 절의 데이터 정렬 후 원하는 일부 데이터만 쉽게 출력할 수 있다.
    ````
    SELECT TOP(2) WITH TIES ENAME, SAL FROM EMP ORDER BY SAL DESC;
    ````

### JOIN

#### EQUI JOIN
- 주의 : FROM 절에 여러 테이블이 나열되더라도 SQL에서 데이터를 처리할 때는 단 두 개의 집합 간에만 조인이 일어난다.
- 테이블의 조인 순서는 옵티마이저에 의해서 결정되고 주요 튜닝 포인트가 된다.

- 하나의 SQL 문장 내에서 유일하게 사용하는 칼럼명이라면 칼럼명 앞에 테이블 명을 붙이지 않아도 되지만,
	<br/> 현재 두 집합에서 유일하다고 하여 미래에도 두 집합에서 유일하다는 보장은 없다.
    <br/> 향후 발생할 오류를 방지하기 위해 유일한 칼럼도 출력할 칼럼명 앞에 테이블명을 붙여서 사용하는 습관 권장
    ````
    예제] 선수 테이블과 팀 테이블에서 선수 이름과 소속된 팀의 이름을 출력하시오.

    SELECT PLAYER.PLAYER_NAME 선수명, TEAM.TEAM_NAME 소속팀명
    FROM PLAYER, TEAM
    WHERE PLAYER.TEAM_ID = TEAM.TEAM_ID;

    또는 INNER JOIN을 명시하여 사용할 수도 있다.
    
    SELECT PLAYER.PLAYER_NAME 선수명,
    TEAM.TEAM_NAME 소속팀명
    FROM PLAYER INNER JOIN TEAM ON PLAYER.TEAM_ID = TEAM.TEAM_ID;
    ````

#### Non EQUI JOIN
````
SELECT E.ENAME, E.JOB, E.SAL, S.GRADE
FROM EMP E, SALGRADE S 
WHERE E.SAL BETWEEN S.LOSAL AND S.HISAL;
````
