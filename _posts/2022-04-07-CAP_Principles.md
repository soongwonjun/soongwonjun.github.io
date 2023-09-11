---
layout: post
title:  "CAP Priciple"
date:  2022-04-07 12:00:00 +0900
categories: cs
tags: cs
description: CAP 정의에 대해 알아보자
---
## CAP Principle?

분산 환경에서는 CAP(Consistency, Availability, Partition Tolerance)의 3가지 중 항상 2가지까지만 만족할 수 있다는 내용이다.  
CAP 원칙에서는 3가지 요소 중 어느 것이 더 중요한지에 대해서 다루지 않는다. CAP 원칙은 각 항목이 어떤 가치를 추구하는지를 이야기 하고 있으며, 우리는 각각의 항목을 이해해고 우리가 구축하려는 시스템에서 어떤 항목을 우선 적용할 것인지를 고려하는 데에 사용하여야 한다.

### Consistency

모든 노드들은 이는 수많은 트랜잭션이 오가는 와중에도 모든 노드들은 같은 데이터 셋을 가지고 있어야 함을 의미한다. 트랜잭션이 발생하여 데이터가 업데이트되면, 모든 노드들은 이를 반영하여야 한다. 이로서 클라이언트는 어느 노드에 접속하는에 관계 없이 항상 동일한 데이터를 조회할 수 있게 된다.

### Availability

클라이언트는 서버의 노드 상태에 관계없이 항상 요청을 보낼 수 있고, 그에 대한 제대로 된 응답을 받을 수 있어야 한다. 설령 일부 노드에 시스템 다운과 같은 문제가 있어도 말이다. 그리고 제대로 된 응답이라는 것은 성공을 의미하기도 하지만, 정상적으로 처리되는 실패 메시지 또한 포함된다. 이는 클러스터가 항상 가용 가능한 상태 아래에 있어야 함을 의미한다.

### Partition Tolerance

하나의 클러스터로 묶인 노드들은 노드 내부의 네트워크 이슈의 이유로 노드 간 커뮤니케이션이 제대로 동작하지 않는다 할지라도, 클러스터 외부에서 볼 때에는 문제없이 동작하고 있어야 한다.
여기서 의미하는 커뮤니케이션은 메시지의 손실 혹은 데이터 송수신의 실패부터 통신의 단절로 인한 클러스터 파티션화가 포함된다.

## Scaling, Partitioning

Scaling과 partitioning은 CAP의 각 요소에 큰 영향을 준다. 인스턴스의 수가 늘어나면 데이터의 일관성을 유지하기 위한 정책이 필요하며, 데이터의 파편화, 동기화에 대해서도 고려해야 하기 때문에 복잡한 처리가 추가로 필요하게 된다. 시스템의 설계자는 CAP 원칙을 고려하고 적용함으로써 시스템의 방향성과 철학을 정립할 수 있다.

### Scaling

인스턴스의 가용 자원을 늘리는 방법을 Vertical scaling이라 한다. 대표적으로 CPU/RAM같은 물리적 자원을 올리는 방법이 여기에 속한다. Vertical scaling은 시스템의 설정/데이터에 변화를 주지 않는다. 또한 물리적 자원에 영향을 받기 때문에 한계가 뚜렷하며 cloud환경에서는 적용하기 어렵다.  
이와는 다르게 인스턴스의 수를 늘리는 방법을 Horizontal scaling, Scale out이라고도 하며, Node를 늘려 데이터 처리를 분산시킨다.

### Partitioning

큰 데이터를 다수의 table 혹은 node에 고르게 분할하여 저장하는 기법이다.  
데이터를 클러스터 혹은 노드 단위로 분할 저장함으로서 데이터 손상률을 최소화하고 데이터 가용성과 응답 속도를 높이는데 목적이 있다.  
데이터의 수(Row)를 나누는 Horizontal Partitioning, 데이터의 구조(Column/Table) 등 정규화된 구조를 나누는 Vertical Partitioning이 있다.

### Sharding

다수의 node에 데이터를 분할 및 그룹화하여 위해 데이터 혹은 특정 키의 해싱 결과인 Shard Key를 사용하여 데이터를 분할 저장하는 기법이다. 노드에 분산된 테이블/컬렉션은 항상 같은 이름, 같은 형상을 가지게 되지만 실제 저장되는 데이터만 다르다. 이름은 같지만 완벽하게 독립된 여러개의 테이블/컬랙션을 가지게 되며 이를 더욱 쉽게 복제, 이동할 수 있게 된다.  
Sharding Key를 작성할 때에는 데이터가 한쪽으로 몰리지 않도록 해주어야 한다. Shard key를 잘못 지정하면 특정 노드에만 과도하게 Read/Write 가 발생하는 Hotspot이라는 현상이 발생하며, 이는 서버의 안정성을 낮추는 문제를 가져온다.
Sharding을 도입할 때에는 아래 2가지를 반드시 고려해야 한다.

- Sharding의 도입으로 테이블/컬랙션간의 Join이 매우 어려워진다.
- 데이터 도메인 단위의 auto increment, last updated, last inserted와 같은 값은 샤드별로 다를 것이다.

### Horizontal Partitioning과 Sharding의 차이점

Sharding은 분산 환경에서의 데이터 분할이 전제된다. 같은 형상의 데이터가 여러 노드에 걸쳐 존재하게 된다.  
Horizontal Partitioning은 하나의 node/instance 내에서의 데이터 분할 및 저장을 의미하기 때문에 Sharding과는 다르다. 하나의 인스턴스 내에서 좀 더 명확하거나 그룹화된 테이블/컬랙션 이름을 가지며(계정 테이블을 고객계정/관리자계정으로 나누는것과 같은), index 혹은 table의 크기를 줄이려 하거나, 명확한 데이터 그룹화를 목적으로 진행한다.

## CA/AP/CP와 DBMS

### CA

RDBMS가 여기에 속한다. RDBMS에서 데이터는 테이블을 사용한 정규화를 진행하며, ACID를 엄격히 지키는 트렌젝션을 통해 제어된다. 때문에 Vertical scaling은 가능해도, Horizontal scaling은 불가능하다. 또한 Master-Slave 환경에서 Master 노드에서 장애가 발생했을 때, Slave 노드가 작업을 대행하여 가용성을 확보한다.

### AP

AP는 가용성을 우선시하기 때문에 분산된 노드간의 데이터 오차를 허용한다. 때문에 데이터 간의 일관성은 보장하지 못하지만, 최종 데이터의 일관성만큼은 보장한다. 때문에 쓰기 작업이 많은 경우에 선호할 수 있다.
예를 들어 Dynamo와 Cassandra는 peer-to-peer 네트워크 시스템을 적용하고 있다. 다수의 노드들이 하나의 클러스터를 이루고 있으며 유저는 어느 노드에 접근하여도 데이터의 조회 및 수정이 가능하다. 하지만 ring 형태로 동작하는 cassandra에서는 하나의 데이터 조작이 모든 노드에 전파될 때 까지는 시간이 걸리며, 이 때 데이터의 일관성이 깨지고, 이를 보완하기 위해서 데이터 버저닝을 지원한다.
![Cassandra_peer-to-peer](/images/20220407/cassandra.png#center)

### CP

CP는 주 역할을 맡는 primary node와 primary node의 replicas인 secondary node로 구성되며, client와의 통신은 primary node가 맡는다.
예를 들어 MongoDB에서 다수의 secondary node replicas는 서로 heartbeat로 상황을 주고받을 수 있으며, primary node에서 secondary node로의 replication을 진행하여 데이터의 일관성을 유지한다. 또한, primary node가 접속 불가능한 상황일 때에는 secondary node가 primary node 역할을 처리한다. 이로서 mongodb는 높은 가용성을 제공한다.
![Mongodb_replicas](/images/20220407/mongodb.png#center)

## Reference

- [Wiki, Shard](https://en.wikipedia.org/wiki/Shard_(database_architecture))
- [Wiki, Partitioning](https://en.wikipedia.org/wiki/Partition_(database))
- [GeeksForGeeks, The CAP Theorem in DBMS](https://www.geeksforgeeks.org/the-cap-theorem-in-dbms/)
- [Apache, Cassandra dynamo](https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html)
- [MongoDB, Replicas](https://www.mongodb.com/docs/manual/replication/)
