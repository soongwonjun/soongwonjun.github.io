---
layout: post
title:  "Cassandra와 MongoDB의 인덱스에 대해서"
date:  2025-04-20 01:00:00 +0900
categories: cs
tags: cs dbms
---

# 서론에 비교를 담아서

|            | MongoDB | Cassandra   |
|------------|---------|-------------|
| CAP 특성 | CP | AP |
| 인덱스 구조 | B-Tree  | LSM Tree + SSTable |
| 인덱스 정렬 단위 | 전체 Collection(Global level) | SSTable |
| 데이터 구조 | BSON | Column-Based(Wide-column Store) |
| 데이터 분산 | Sharding | Partition Key |
| 데이터 저장 | 디스크 내에서 index를 기준으로 저장 | SSTable 내에서 Index를 기준으로 저장 |
| 데이터 특성 | Mutable | Immutable |
| Transaction | Document에 대한 Transaction 있음 | 일부만 지원* |
| Insert | 느림(위치 찾아서 저장 후 재정렬) | 빠름(MemTable에 저장 후 머지) |
| Update | 기존 데이터 변경 | 새 Row가 생성됨 |
| Select | 다양한 필터 조건(Equal, Range)에 대응되어 있음 | Parition Key 기반은 빠름, SAI는 비교적 느림 |

- Casssandra의 트랜잭션은 단일 row에 대한 처리 혹은 단일 파티션 내의 Batch정도만을 지원함

# MongoDB의 Index에 대해서

MongoDB는 다양한 인덱스를 제공한다.
- Default Index
- Single Field Index
- Compound Index
- Multikey Index
- Geospatial Index
- Text Index
- Hashed Index
- Clustered Index

MongoDB에서의 모든 인덱스는 별도의 B-Tree로 구성되어 있으며, 데이터와는 별도의 자료구조로 저장된다.  

## Default Index

`_id` 필드 인덱스. 모든 document에 생성되며 제거가 불가능하다. 이 `_id` 컬럼은 `ObjectId`라는 BSON이며, timestamp, 컴퓨터ID, 프로세서ID를 사용하여 고유함을 유지한다.

## Sinle Field Index

단일 field를 사용하여 생성한 인덱스

## Compound Index

다수의 field를 사용하여 생성한 인덱스

## Multikey Index

배열에 적용되는 인덱스. MongoDB에서는 document에 배열이 생성되면 자동으로 만들어진다.

## Geospatial Index

지리 공간을 처리하기 위해 특별히 만들어진 인덱스. GeoJSON 혹은 좌표(위도, 경도의 쌍) 정보를 저장하는 특별한 자료구조를 사용하여 기하학 계산을 지원한다.

## Text Index

텍스트 필드를 위한 인덱스

## Hashed Index

샤딩을 위한 해시된 값을 사용해 만드는 인덱스. `ObjectId` 혹은 `timestamp`같은 단순한 값을 사용하며, multikey는 지원하지 않는다. `convertShardKeyToHashed()` 함수를 사용해서 실제 해시된 값을 찾아낼 수 있다.  
데이터를 해시된 데이터로 저장하므로 데이터 위치 특정이 매우 쉽다. 때문에 `Equal`검색에 매우 강력하다. 하지만 이 Hashed Index는 정렬되어 있지 않기에 범위 검색에는 적합하지 않다. 무작위로 분포되는 이유는 가능한 한 고르게 분포 되어야 HotSpot을 방지할 수 있기 때문이다.

## Clustered Index

MongoDB가 디스크에 데이터를 저장할 때 인덱스 순서대로 저장될 수 있도록 하는 인덱스. Mongodb가 구성하는 Index의 순서 그대로 디스크에 저장되기 때문에 물리 디스크에서 데이터를 가져올 때 상당히 빠른 처리가 가능하다. 단 하나의 document에서 하나의 clustered index만 지정할 수 있다.  
이 Clustered Index는 Sharding과는 다르다. Sharding은 노드의 분산을 의미하지만, Clustered Index는 샤딩된 후 개별 노드에 대해 적용된다. 때문에 Clustered Index를 쓰면서도 정렬된 Hashed Index를 사용하기 위해서는 Ranged Sharding을 사용해야 하는데 이는 HotSpot으로 인한 resharding 문제가 있어 성능에 이슈가 있을 수 있다.

# Cassandra의 Index 에 대해서

Cassandra의 인덱스는 크게 3가지가 있다.

- Primary Index
  - Cassandra의 Partition Key. 가장 빠르고 효과적임
- Storage-attached indexing (SAI)
  - non-partition column을 위한 인덱스
  - SSTable을 사용해서 데이터를 저장함
  - cassandra 5.0부터 지원
- Secondary indexing (2i)
  - Cassandra의 각 노드에 숨겨진 테이블로서 존재하는 로컬 인덱스
  - Primary Index(Partition Key)와 함께 써야만 함
  - 성능상 이슈가 발생히가에 cassandra에서는 이 인덱스보다는 SAI를 쓰도록 가이드하고 있음

기본적으로 Cassandra는 Partition Key를 기반으로 데이터를 저장하므로 이 정보를 가진 Primary Key가 가장 빠르다. 하지만 5.x 부터 지원하는 SAI는 유연성에서 Primary Key를 대체할 수 있는 경우가 있다.  
Primary Index는 Hash로 저장되어 있고, 이 값은 equal/in에 대해 상당히 강력한 구조이다. 하지만 Between, GTE와 같은 범위 구조에 대해서는 사용할 수 없기 때문에 이러한 부분을 SAI가 보완해준다.

## Primary Index

Key 기반 인덱스
테이블에는 하나의 row를 식별하기 위해 1개의 Primary Key를 정의해야 하고, 보통은 1개 이상의 컬럼을 사용한다.
또한 Cassandra는 row 단위로 데이터를 찾는것이 아닌 partition 단위로 데이터를 찾는 구조이다.
때문에 Primary Key는 Partition Key와 Clustering Column으로 나뉘어 진다.

- Partition Key
  - 테이블에 지정하는 컬럼. 어떤 Partition을 사용할지를 결정한다. 이 Partition은 하나의 Node에 모이게 된다.
  - 즉 어느 node에 저장할지를 결정한다는 의미이기도 하다.
- Clustering Column
  - Primary key를 정의할 때 Partition Key 뒤에 이어지는 컬럼으로, Partition 내의 정렬된 순서를 지정한다.

Cassandra는 이 데이터를 Hash로 만든 정렬된 데이터로 저장한다. 때문에 Primary 키를 사용하면 데이터가 저장된 노드를 바로 식별하고, 정렬된 범위 안에서 즉각적으로 데이터를 찾아내므로 굉장히 빠르다. 때문에 `Equal`/`In` 검색같이 위치가 특정되는 검색에 매우 강하다.

### Partition

카산드라는 데이터를 Partition Key 단위로 묶어두고, 이 데이터 묶음을 모두 같은 노드에 저장한다.
카산드라는 분산 시스템이여서, 데이터가 어디에 저장되는지를 알아야 데이터를 올바르게 찾고, 저장할 수 있다.
그리고 Partition Key를 통해 노드를 식별하면 효율적이고, 빠르다.
때문에 Partition Key의 정의에 따라 카산드라의 전체 효율이 결정된다.

### Primary Index의 예제

```sql
CREATE TABLE users (
  user_id UUID,
  created_at TIMESTAMP,
  name TEXT,
  PRIMARY KEY (user_id, created_at)
);
```

- 3개의 컬럼을 가진 users 테이블
- user_id는 Partition Key
- created_at은 Clustering Column

데이터는 아래와 같이 저장된다

```
partition[userid=1001]
  └─ [created_at=2023-01-01] name = "Alice"
  └─ [created_at=2023-01-02] name = "Bob"
  └─ [created_at=2023-01-03] name = "Charlie"
partition[userid=1002]
  └─ [created_at=2023-01-01] name = "David"
  └─ [created_at=2023-01-02] name = "Eve"
```

user id로 node와 partition이 결정되고, 그 안에서 created_at 으로 정렬되어 있음
하지만 created_at 만으로 검색하는 경우, partition key를 알 수 없기에 full-scan을 해야한다.

## Storage-attached indexing (SAI)

- 분산 환경을 위한 높은 확장성을 가진 인덱스
- 컬럼별로 따로 데이터를 저장하는 방식을 사용한다.
- MemTable을 사용하여 SSTable을 인덱싱하고, 데이터가 쓰여질 때 이 MemTable을 갱신하여 인덱스를 유지한다.
- read 시 사용과는 과정
  - cassandra는 query가 들어오면 적용 가능한 index를 검색한다.
  - 먼저 primary key를 사용해 partition을 찾고, partition에 해당되는 MemTable을, 결과가 없다면 sstable을 조회한다.
  - 이 작업은 병렬로 처리되고, 결과를 취합하여 response를 만들어 결과를 떨군다.
- write의 과정
  - 데이터 변경사항이 먼저 MemTable에 반영된다.
  - MemTable이 flush될 때 sstable에 반영된다.
  - sstable에 반영될 때 컬럼 단위 index가 생성된다. 이 인덱스가 SAI이다.
- 범위 검색이 가능한 구조
  - Primary Index는 `Equal`/`IN`이 아니면 제대로된 인덱스 검색이 불가능하다. 하지만 SAI는 유연한 구조를 위해 만들어졌기 때문에 `BETWEEN`같은 조건을 사용한 범위 필터링이 가능하다.

## Secondary Indexing

Partition Key가 아닌 다른 컬럼에 인덱스를 만들어서 사용하는 방식
간단하게 인덱스를 구성할 수 있으나, 모든 노드에 개별 테이블로서 존재하기에 성능 이슈가 발생한다. 노드를 특정할 수 없기에 모든 노드를 뒤져야해서 full-scan 처럼 작동하기 때문이다.
매우 작은, 혹은 단일 노드에서는 사용할 수도 있으나, 클러스터 환경이나 큰 데이터를 다루어야 한다면 사용하지 않는 편이 좋다.

```sql
CREATE INDEX ON users(name);
```

## 번외

### Sharding이 뭘까?

데이터를 특정 기준(Hashed된 키)를 사용하여 분산하여 데이터베이스의 수평적 확장(Scale-out/Horizontal partition)을 가능케 한다.

- 장점
  - 읽기/쓰기 처리량 증가
  - 매우 쉬운 저장용량 증가
  - 고가용성. 샤딩을 구현하기 위해서는 데이터의 분산 및 복제를 전제로 처리되기 때문에 고가용성이 보장된다.
- 단점
  - 데이터 조회의 오버헤드. 데이터 조회 시 마다 이를 조율할 수 있는 기능, 데이터를 취합할 수 있는 기능이 수행되어야 한다.
  - 관리의 복잡성. 관리 대상의 증가, 그리고 데이터의 분산 및 복제를 위한 처리, 이를 위한 백업 플랜 등 많은 기능이 요구된다.

### LSM (Log Structured Merge Tree), MemTable, SSTable이 뭘까?

key-value 쌍을 저장장치에 기록하기 위한 자료구조. 쓰기 성능에 최적화되어 있다. 메모리와 디스크 모두 쓰기 최적화를 위해 데이터를 메모리 -> 디스크 순으로 저장하는 과정을 거치며, 이 과정에 있어 불필요한 데이터(deprecated된 데이터)는 디스크에 반영하지 않는다. 때문에 항상 메모리와 디스크의 데이터가 같다는 보장이 없다.  
LSM 은 MemTable, CommitLog, SSTable이라는 3개의 구성 요소를 가진다.

### MemTable이 뭘까?
- cassandra에서 데이터를 임시 저장하기 위한 메모리 내의 가상 테이블
- MemTable은 임시 테이블이어서, 이 데이터가 flush 되기 전에 서버가 죽으면 MemTable의 데이터는 소실된다.
- 하지만 MemTable에 저장될 때 commitlog에 기록되기 때문에 이를 사용해서 데이터를 복구한다.
- 데이터가 기록되는 과정
```
user - (insert / update) -> cassandra
                             └─ write commitlog (disk)
                             └─ write MemTable             - (flush) -> sstable
```

### SSTable 이 뭘까?
- Sorted String Table의 약자
- ScyllaDB, Cassandra같은 NoSQL DB에서 사용되는 데이터 저장을 위한 파일 포멧
- 빠른 읽기를 위해 정의된 순서대로 데이터를 정렬해서 저장하며, 이 데이터들은 불변 데이터이다.
- 새로이 갱신되는 데이터는 새로운 MemTable에 저장되고, flush될 때 새로운 SSTable로 저장된다.

### 왜 Cassandra는 SSTable이 여러개 생길까?

이는 카산드라의 데이터 저장 방식에 기인한다.
- SSTable은 immutable한 구조로, 한번 생성되면 수정할 수 없다.
- 새로 업데이트하는 내용도 계속 추가가 될 뿐, 기존 데이터가 바뀌지는 않는다.
- SSTable은 MemTable이 flush될 때 생성되는데, 이 타이밍에 따라 여러개의 SSTable이 생성될 수 있다.
- insert가 많아도 동일 MemTable에 쌓이면 1개의 SSTable이 생성되겠지만, 여러개의 MemTable이 사용되기에 꼭 1개만 생성되지는 않는다.
- 한번 MemTable이 flush 된 이후에 기존 데이터가 업데이트 되면 다른 MemTable에 데이터가 저장되기에 새 SSTable이 생성된다.

시나리오는 아래와 같다.
```sql
INSERT INTO users (id, name, age) VALUES (1, "alpha", 30);   -- MemTable 1

-- MemTable full 에 의해 flush → SSTable A 생성

INSERT INTO users (id, name, age) VALUES (2, "beta", 31);    -- MemTable 2
INSERT INTO users (id, name, age) VALUES (3, "challie", 22); -- MemTable 2

-- MemTable flush → SSTable B 생성
```

이러한 문제로 인하여 카산드라는 `compaction`이라는 과정을 통해 여러개의 SSTable을 하나로 합쳐서 관리한다.
- compaction은 여러개의 SSTable의 데이터를 하나의 완전한 row로 만드는 작업
- partition key를 기준으로 정렬 및 병합을 진행하여 1개의 SSTable을 생성한다.
- 오래된 데이터를 담은 SSTable은 진행중인 읽기 과정이 끝나면 삭제된다.

### LSM Tree vs B-Tree를 비교해봅시다.

|  |LSM Tree | B-Tree|
|--|---------|------|
| 설계 목적 | 쓰기 성능 최적화 | 읽기 성능 최적화 |
| 트리 구조 | 다단계 파일 + 병합(compaction) 필요 | 균형 트리 |
| 읽기 구조 | Memtable 먼저 읽은 후 여러 SSTable을 병합 읽기 | B-Tree 탐색 후 디스크에서 읽기 |
| 쓰기 구조 | 메모리에 작성 후 디스크로 옮겨감 | 디스크에 바로 쓰며 노드 분할/병합 수행 |
| 삭제 구조 | 메모리에 삭제마크 후 Compaction 시 삭제 | 직접 해당 노드에서 키 제거 |
| 업데이트 방식 | 없음. 추가만 가능 | 직접 수정 가능 |
| 정렬 유지 방식 | flush 및 병합 시 정렬된 SSTable 생성 | 트리 내에서 정렬된 상태 유지 |

# 참고
- [Cassandra, Index에 대해서](https://cassandra.apache.org/doc/5.0/cassandra/developing/cql/indexing/indexing-concepts.html)
- [ScyllaDB, SSTable에 대해서](https://www.scylladb.com/glossary/sstable/)
- [ScyllaDB, LSM Tree에 대해서](https://www.scylladb.com/glossary/log-structured-merge-tree/)
- [MongoDB, Index에 대해서](https://www.mongodb.com/ko-kr/docs/manual/indexes/)