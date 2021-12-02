# NoSQL과 MongoDB

NoSQL와 MongoDB에 대해 알아보자 !
<br><br>

## NoSQL이란?
NoSQL은 **Not Only SQL**, 즉 SQL을 사용하는 RDBMS가 아닌 데이터베이스를 의미한다.  
대표적인 `RDBMS`로는 MySQL, Oracle, PostgreSQL 등이 있고, `NoSQL` 진영에는 MongDB, Redis, HBase 등이 있다.

<br><br>


### 왜 RDBMS가 아닌 `NoSQL`를 사용할까?  

보통 서비스들은 `RDBMS`를 기본으로 사용했지만 소셜 SNS를 비롯한 온라인 서비스들이 등장하면서 비정형 데이터를 쉽게 담아서 처리할 수 있는 DB에 대한 수요가 생겼다.

또한 클라우드, 분산형 컴퓨팅이 주목받고 있는데 SQL은 이에 적합하지 않다. SQL은 강력한 정보 저장 수단이지만 다양한 환경과 분산 컴퓨팅 추세에 따라 SQL의 약점이 나타나게 된 것이다.

그래서 이러한 약점을 보완할 수 있는 `NoSQL`이 각광을 받게 되었다. 

<br>

`NoSQL`은 기존의 관계형 데이터베이스 시스템의 주요 특성을 보장하는 **ACID를 지키지 않고, BASE를 선택했다.** 뛰어난 확장성이나 성능 등 특성을 갖는 비관계형 데이터 베이스이다.

이는 단순 검색 및 추가 작업에 있어서 매우 최적화된 키 값 저장 기법을 사용하며 응답속도나 처리 효율등에 있어서 매우 뛰어난 성능을 나타낸다.

<br><br>

### ACID와 BASE

#### ACID
DB 트랜잭션이 안전하게 수행됨을 보장
- 원자성(**A**tomicity)
  - 트랜잭션과 관련된 작업이 부분적으로 실행되다가 중단되지 않는 것을 보장하는 능력
- 일관성(**C**onsistency)
  - 트랜잭션이 실행을 성공적으로 완료하면 언제나 일관성 있는 DB 상태로 유지하는 것을 의미
- 고립성(**I**solation)
  - 트랜잭션을 수행 시 다른 트랜잭션의 연산 작업이 끼어들지 못하도록 보장하는 것을 의미
- 지속성(**D**urability)
  - 성공적으로 수행된 트랜잭션은 영원히 반영되어야 함을 의미한다.
  
#### BASE
ACID와 대조적으로 가용성과 성능을 중시하는 분산 시스템의 특성
- **B**ase **A**vailable

<br><br>

### NoSQL과 RDBMS의 차이

#### NoSQL
- 테이블 관의 관계(JOIN)이 없다
- RDBMS보다 복잡하지 않아서 대용량의 데이터를 저장, 관리할 수 있다
- 스키마가 없어 데이터 저장이 유연하고 자유롭다
- 스키마가 없어 명확한 데이터 구조를 보장하지 않는다
- 수평적 확장이 RDBMS보다는 쉬움
- 데이터 중복이 발생할 수 있고 데이터 변경 시 모든 컬렉션에서 수정을 수행해야 한다


#### RDBMS
- 데이터의 분류, 정렬, 탐색 속도가 비교적 빠르다
- SQL이라는 구조화 된 질의를 통해 데이터를 다룰 수 있다
- 스키마 규격에 맞춰 데이터를 다뤄야 한다
- 스키마가 정해져 있기 때문에, 명확한 데이터 구조를 보장함(데이터 무결성 보장)
- 데이터를 중복없이 한 번만 저장할 수 있다
- 시스템이 커질 경우 JOIN문이 많은 복잡한 쿼리가 만들어질 수 있다

<br><br>

### NoSQL의 종류

1. Key-Value DB  
    데이터가 Key-Value 쌍으로 저장되고 Key에는 어떠한 형태의 데이터도 담을 수 있다.  
    간단한 API를 제공하는 만큼 속도가 빠르다.  
    ex) Redis, Riak, Amazon Dynamo DB   
      
2. Document DB
    데이터가 Key-Document 형태로 저장된다.  
    Value가 계층적인 형태인 Document로 저장된다. 이는 객체 지향의 객체와 비슷하며 하나의 단위로 취급된다. 즉, 하나의 객체를 여러 테이블에 나눠 저장할 필요가 없어진다.  
    검색에 최적화 되어있고 단점은 사용이 번거롭고 쿼리가 SQL과 다르다.  
    ex) MongoDB, CouthDB  
      
3. Wide Column DB
    Column-family Model 기반의 DB이다  
    이는 키에서 필드를 결정한다. 키가 Row(키 값)와 Column-family, Column-name을 가진다. 연관된 데이터는 같은 Column-family안에 속하며 각자의 Column name을 가진다. 이렇게 저장 된 데이터는 커다란 하나의 Table로 표현이 가능하고 질의는 Row, Column-family, Column-name을 통해 수행된다.  
    ex) HBase, Hypertable  

4. Graph DB  
    데이터를 Node와 Edge, Property와 함께 그래프 구조를 사용하여 데이터를 표현하고 저장하는 DB이다. 

<br><br>

=> `RDBMS`는 DB 구조가 변경될 여지가 없으며 명확한 스키마가 중요한 경우 사용한다. 또한 관계를 맺고 있는 데이터가 자주 변경되는 시스템에 적합하다.  
=> `NoSQL`는 정확한 데이터 구조를 알 수 없는 경우나 Scale-out이 가능하므로 막대한 데이터를 저장해야 하는 시스템에 적합하다.

<br><br><br>

## mongoDB
`mongoDB`는 `NoSQL` DB로 Document 기반으로 구성되어 있다. 또한 오픈 소스이기 때문에 무료로 이용이 가능하다.

Database > Collection > Document > Field 계층으로 이루어져 있으며 Document는 RDBMS의 Row에 해당한다. 


<br><br><br>

## Ref
https://aws.amazon.com/ko/nosql/  
https://www.samsungsds.com/kr/insights/1232564_4627.html    
https://kciter.so/posts/about-mongodb  
https://pythontoomuchinformation.tistory.com/528   
https://atin.tistory.com/624  