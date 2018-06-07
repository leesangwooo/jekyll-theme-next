

#### SQL과 NoSQL 용어 비교
||SQL|NoSQL|비고|
|-|-|-|-|
|1|table|collection||
|2|Column|BSON Field||
|3|Row|BSON Document||
|4|Primary Key|_ID Field||
|5|RelationShip|Embedded & Linking||

####MongoDB 문법
- MongoDB 클라이언트 인스턴스 생성 : `mongod --dbpath d:\my\workspace\`
<hr>
#####컬랙션
- 컬렉션 생성하기 : `db.createCollection("collection_name", { capped : false, size : 8192 })`
	- capped : 해당 공간이 모두 사용되면 다시 처음부터 재사용할 수 있는 데이터 구조를 생성
	- size : 해당 collection의 최초 생성크기 지정

- 생성된 컬렉션 확인하기 : `show collections`

- 해당 컬렉션 삭제하기 : `db.collection_name.drop()`

- 컬렉션의 현재 상태 확인하기 : `db.collection_name.validate()`

- 해당 컬랙션의 이름 변경하기 : `db.collection_name.renameCollection("renamed_collection")` 
<hr>
#####도큐먼트
- 해당 컬렉션의 document를 추가하기 : `db.collection_name.insert({ field_name_01 : value, field_name_02 : value });`

- 해당 컬렉션의 document를 수정하기 : `db.collection_name.update({ _id_field_검색조건 : 1 }, { $set: {new_field_name : "string_value"} });`

- `.save()` : 필드명으로 도큐먼트를 조회하여, $set:{추가할 JSON}를 '덮어씌우는 방식으로' 도큐먼트를 갱신한다.
	\* save, insert, update의 차이 [참고. 블로그 '아는만큼 보여요'](http://squll1.tistory.com/entry/mongodb-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%82%BD%EC%9E%85-%EC%B6%9C%EB%A0%A5-%EC%88%98%EC%A0%95)

- 해당 컬렉션의 전체 도큐먼트 조회하기 : `db.collection_name.find();`
	- 정렬하기 : `.sort({정렬방법})`
	- 정리하여 나타내기 : `.pretty()`

- 해당 컬렉션의 해당 도큐먼트 삭제하기 : `db.collection_name.remove({_id_field_name: value});`

####Two Task Rollback
NoSQL의 DBMS는 보다 빠른 데이터 쓰기와 읽기를 위해 도큐먼트의 변경사항을 자동으로 커밋(Auto Commit)한다. 하지만, 의미적으로 또는 기능적으로 연결된 collection들 간, 앞서 실행한 트랜잭션과 이후 실행할 트랜잭션 간에 커밋 시점의 차이로 인해 `데이터의 일관성`이 무너지는 경우가 있다. 
> A의 계좌에서 100만원을 인출하여 B의 계좌에 송금하는 경우, A의 잔액에서 100을 빼는 트랜잭션과 B의 잔액에서 100을 더하는 트랜잭션이 동시적으로 이루어져야 한다. 만약 두 트랜잭션 간에 커밋 시점에 차이가 나는 경우 데이터의 일관성이 무너진다.

**실제 트랜잭션은 끝났지만 명시적으로 트랜잭션을 제어하는 구분자를 이용하여 **
- 거래 중 상태를 'pendingTransactions' 필드로 식별하고자 함. 
``` 
> db.accounts.save({name:"Jimm", balance: 2500, pendingTransactions:[]})
> db.accounts.save({name:"King", balance: 1300, pendingTransactions:[]})
```

- 트랜잭션을 관리하기 위한 'transactions' 컬랙션. 트랜잭션 대상인 두 컬랙션을 source, destination 칼럼에 저장하고, 트랜잭션 처리를 통해 변환할 값의 크기를 value에 저장, 그리고 현재 트랜잭션 상태를 식별할 수 있는 'state' field를 "initial"로 초기화한다.
```
> db.transactions.save({source: "Jimm", destination: "King", value: 100, state: "initial"});
```
- 컬랙션에서 state field의 값이 "initial"인 도큐먼트를 변수 t에 담는다.
변수 t의 _id field 값으로 트랜잭션 대상을 찾아 현재 상태를 "pending"으로 변경한다. 
```
> t = db.transactions.findOne({state: "initial"});
> db.transactions.update({_id: t._id}, {$set: {state: "pending"}});
```

- accounts 컬랙션에 name 필드의 값이 t의 source이고, pendingTransactions의 값이 t의 _id와 같지 않은 경우
해당 도큐먼트의 balance 값을 t의 value인 100만큼 음의 방향으로 inc(rease)하고,
- accounts 컬랙션에 name 필드의 값이 t의 destination이고, pendingTransactions의 값이 t의 _id와 같지 않은 경우
해당 도큐먼트의 balance 값을 100만큼 inc(rease)한다.
```
> db.accounts.update({name:t.source, pendingTransactions: {$ne: t._id}},
	{$inc: {balance: -t.value}, $push: {pendingTransactions: t_id}});
> db.accounts.update({name:t.destination, pendingTransactions: {$ne: t._id}},
	{$inc: {balance: t.value}, $push: {pendingTransactions: t_id}});
```

- 트랜잭션 상태를 canceling으로 변경
```
db.transactions.update({_id: t._id}, {$set: {state: "canceling"}});
```

```
> db.account.update({name:t.source, pendingTransactions: t._id}, {$inc: {balance: }});
```