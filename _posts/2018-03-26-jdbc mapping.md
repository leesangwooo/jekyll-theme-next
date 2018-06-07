JDBC ResultSet Mapping하기(1)
Map과 VO의 사용

####JDBC ResultSet
***
JDBC의 쿼리 결과는 int 또는 ResultSet의 형태로 반환된다.
- `int executeUpdate()` : 갱신을 위한 쿼리 실행 메소드. 갱신한 행의 갯수를 return한다.

- `ResultSet executeQuery()` : 조회를 위한 쿼리 실행 메소드.

&nbsp;&nbsp;이때 `ResultSet`은 select 쿼리를 사용하여 조회한 레코드 값들을 테이블 형태로 저장하며, 조회한 칼럼명과 , 칼럼수, 칼럼 타입 등의 메타데이터를 갖고있다. 
[참고.](http://nyhooni.tistory.com/71)

####JDBC ResultSet을 Mapping하기
***
&nbsp;&nbsp;조회한 데이터를 효과적으로 사용하기 위해 ResultSet을 Collection 또는 VO, iBatis 등을 활용해 매핑할 수 있다.
아래는 가장 기본적인 방법인 Collection과 VO를 활용한 매핑에 대해 정리하였다.
<table>
    <thead>
        <tr>
            <th></th>
            <th>Map</th>
            <th>VO (Value Object)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>장점</td>
            <td>VO에 비해 사용이 간편하다.
            유연한 데이터 처리가 가능하다.
            </td>
            <td>객체 지향적인 방식으로(단일 책임),
            코드의 재활용과 확장가능성이 유리하다.
            타입 안정성이 높다.</td>
        </tr>
        <tr>
            <td>단점</td>
            <td>타입 안정성이 보장되지 않는다.
            (key 값을 하드코딩하는 경우,
            오타 등의 오류가 발생할 가능성이 있다.)
            </td>
            <td>Map에 비해 패키지 디자인의 소요가 있고
            구현이 어렵다.</td>
        </tr>
        <tr>
            <td>사용</td>
            <td>동적인 데이터 처리에 유용
            </td>
            <td>프라퍼티의 변경 수요가 적은
            정적인 데이터 처리에 유용 </td>
        </tr>
        <tr>
            <td>참고</td>
            <td>생활코딩 - [java collection framework]( https://opentutorials.org/course/1223/6446)</td>
            <td>개발자블로그 - [DAO/VO/DTO란?](http://genesis8.tistory.com/214)</td>
        </tr>
    </tbody>
</table>

####코드분석
***
예) DB에 저장된 BUYER 테이블을 SELECT하기 위해 DAO에서 JDBC를 구현한 메소드
- Map 과 VO는 요구하는 상황에 맞게 선택적으로 사용한다.

```java
@Override
public List<BuyerVO> selectBuyerList(Map<String, Object> pMap) {
    List<BuyerVO> buyerList = new ArrayList<BuyerVO>();
    StringBuffer sql = new StringBuffer();
    sql.append(" SELECT BUYER_ID, BUYER_NAME, BUYER_MAIL, BUYER_LGU, BUYER_ZIP	");
    sql.append(" FROM BUYER														");
    try(
        Connection conn = ConnectionFactory.getConnection();
        PreparedStatement pstmt = conn.prepareStatement(sql.toString());
    ){
   	 ResultSet rs = pstmt.executeQuery();
   	 /////////////////////////////////////////////////////////////////////////////
   	 //Map을 사용하여 ResultSet의 MetaData 매핑
        int colCount = rs.getMetaData().getColumnCount();
        String[] headers = new String[colCount];
        for(int i=1; i <= headers.length; i++){
            headers[i-1] = rs.getMetaData().getColumnName(i);
        }
        // **CALL BY REFERNCE**
        pMap.put("headers", headers); 
        
		/////////////////////////////////////////////////////////////////////////////
        //VO를 사용하여 ResultSet의 레코드 값을 매핑
        while(rs.next()){
            BuyerVO buyer = new BuyerVO();
            buyer.setBuyer_id(rs.getString("BUYER_ID"));
            buyer.setBuyer_name(rs.getString("BUYER_NAME"));
            buyer.setBuyer_mail(rs.getString("BUYER_MAIL"));
            buyer.setBuyer_lgu(rs.getString("BUYER_LGU"));
            buyer.setBuyer_zip(rs.getString("BUYER_ZIP"));
            buyerList.add(buyer);
        }
        return buyerList;
        /////////////////////////////////////////////////////////////////////////////
    }catch(SQLException e){
        throw new CommonException(e); // custom runtime exception
    }

}
```





