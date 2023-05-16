### Index란?

추가적인 쓰기 작업과 저장 공간을 활용하여 데이터베이스의 검색 속도를 향상시키기 위한 데이터 구조이다.

### Index를 왜 써야할까?

**PLAYER**

| id | name | team_id | backnumber |
| --- | --- | --- | --- |
| 1 | Hunhui | 1 | 13 |
| … | … | … | … |
| 1000000 | Minsoo | 105 | 7 |

위와 같이 데이터가 100만건 있는 player 테이블이 있다.

여기서 아래와 같이 name이 “Minsoo”인 player를 찾는 쿼리가 있다고 해보자.

```sql
SELECT * FROM player WHERE name = 'Minsoo';
```

만약, name에 index가 걸려있지 않을 경우 100만건의 데이터를 전부 찾아보는 **full scan**을 하게 되고 이때, 시간 복잡도는 `O(N)`이 된다. (N은 데이터 개수)

하지만, name에 B-Tree 기반 index가 걸려있을 경우 시간 복잡도는 `O(logN)`이 된다.

→ **Index는 조건을 만족하는 튜플들을 빠르게 조회, 정렬(order by), 그룹핑(group by)하기 위해 사용한다.**

### Index 생성 방법

mysql 기준으로  Index를 생성하는 방법은 아래와 같다.

```sql
// index 생성
CREATE [UNIQUE] INDEX index_이름 ON table_이름 (대상_attribute[, ...]);

// 테이블 생성과 함께 index 생성 가능
CREATE TABLE player(
	id INT PRIMARY KEY,
	name VARCHAR(20) NOT NULL,
	team_id INT,
	backnumber INT,
	INDEX player_name_idx (name), // index 이름 생략 가능
	UNIQUE INDEX team_id_backnumber_idx (team_id, backnumber)
)
```

인덱스는 `일반 인덱스`와 `유니크 인덱스`로 나눌 수 있다.

- 일반 인덱스 생성 시
    
    ```sql
    CREATE INDEX player_name_idx ON player (name);
    ```
    
- 유니크 인덱스 생성 시
    
    ```sql
    CREATE UNIQUE INDEX team_id_backnumber_idx ON player (team_id, backnumber);
    ```
    

일반 인덱스는 중복된 값을 가질 수 있고, 유니크 인덱스는 중복된 값을 가질 수 없다.

위 두 인덱스를 예시로 들면, 이름(name)같은 경우는 동명이인이 있을 수 있기 때문에 일반 인덱스로 생성한 모습이고, 같은 팀에 특정 등번호를 가진 사람은 유일하기 때문에 유니크 인덱스로 생성한 모습이다.

유니크 인덱스는 중복된 값을 허용하지 않기 때문에 데이터 삽입, 수정 작업 시 일반 인덱스에 비해 더 많은 시간이 걸릴 수 있다.

추가적으로..

- PK의 경우는 index가 자동 생성된다.
- 여러 attribute(아래의 경우는 team_id 와 backnumber)로 생성한 index를 “복합 인덱스”라고 한다.
    
    ```sql
    CREATE UNIQUE INDEX team_id_backnumber_idx ON player (team_id, backnumber);
    ```
    
- 아래의 쿼리로 테이블에 걸려있는 index를 확인할 수 있다.
    
    ```sql
    SHOW INDEX FROM 테이블_이름;
    ```
    

### B-Tree 기반 Index 동작 방식

아래 예시용 인덱스, 테이블을 이용하여 index의 동작방식을 살펴본다.

**1번 인덱스 (team_id 단일 인덱스)**

| team_id | ptr |
| --- | --- |
| 1 | … |
| 1 | … |
| 2 | … |
| 2 | … |
| 3 | … |
| 4 | … |
| 5 | … |
| 5 | … |
| 5 | … |
| 7 | … |
| 12 | … |

**2번 인덱스 (team_id, backnumber 복합 인덱스)**

| team_id | backnumber | ptr |
| --- | --- | --- |
| 1 | 1 | … |
| 1 | 2 | … |
| 2 | 1 | … |
| 2 | 2 | … |
| 3 | 1 | … |
| 4 | 1 | … |
| 5 | 1 | … |
| 5 | 2 | … |
| 5 | 3 | … |
| 7 | 1 | … |
| 12 | 1 | … |

index는 기본적으로 index에 해당하는 attribute의 값과, 해당 attribute가 속한 튜플의 위치를 가리키는 포인터를 가지고 있고, attribute의 값으로 정렬되어 있다.

2번 인덱스 같은 복합 인덱스의 경우는 인덱스의 순서에 따라 team_id 기준으로 정렬 후 같은 team_id를 가진 튜플들은 backnumber 기준으로 정렬되어 있다.

```sql
WHERE team_id = 5 AND backnumber = 3
```

위 인덱스와 player 테이블을 기준으로 위의 조건을 만족하는 튜플을 찾아보자.

- B tree 기반 인덱스이므로 실제로는 이진 탐색이 아니지만, 예시를 들기 위해 이진탐색을 한다고 가정한다.

1. **1번 인덱스 사용 시**
    - 우선 1번 인덱스에서 중간 값(4)를 찾아서 조건과 비교
        
        → 조건의 값(5)보다 작으므로 4이후의 튜플들만 다시 탐색
        
    - 남은 튜플 중 중간 값(5)를 찾아서 조건과 비교
        
        → 조건과 일치, 이어서 해당 튜플의 backnumber가 저장된 테이블에 접근하여 확인
        
        → backnumber는 조건과 맞지 않으므로 pass
        
    - 이어서 남은 값들 중 위쪽의 값은 5보다 크니까 무시하고 남은 값들 확인
    - 또 다른 team_id가 5인 player를 발견, 해당 튜플의 backnumber가 저장된 테이블에 접근
        
        → 조건과 맞지 않으므로 pass
        
    - 남은 범위 중에 team_id가 5인 또 다른 튜플를 발견, 해당 튜플의 backnumber가 저장된 테이블에 접근
        
        → 조건과 일치,  최종적으로 해당 조건을 만족하는 튜플을 가지고있는 테이블에 접근
        
2. **2번 인덱스 사용 시**
    - 1번 인덱스 사용 시와 비슷하지만, 조건(team_id = 5)을 만족하는 튜플 발견 시 backnumber를 확인하기 위해서 다른 테이블에 접근할 필요 없이 같은 블록에 위치한 backnumber를 확인
    - 즉, 1번 인덱스와 달리, 조건을 확인할 때 사용되는 backnumber 컬럼에 대한 데이터가 같은 테이블에 있으므로 직접 player table에 접근하는 횟수가 줄어듬

→ 두 인덱스는 다른 테이블에 직접 접근하는 **디스크 I/O 작업** 횟수의 차이가 있고, 당연히 위의 사례에서는 2번 인덱스(복합 인덱스)를 사용하는 것이 더 효과적이다.

*** 쿼리가 어떤 인덱스를 사용하는 지 알 수 있는 방법**

- Explain 쿼리를 실행하여 key 속성으로 확인할 수 있다. 아래의 경우는 PK로 생성되는 index를 사용했음을 나타낸다.
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/54226b7f-cbcb-4349-9847-9dce9145f72b)
    
- 원래는 Optimizer가 적절한 index를 골라주지만, 가끔 이상한 인덱스를 고를 때가 있다.
- `USE INDEX (인덱스_이름)`(인덱스 권장), `FORCE INDEX (인덱스_이름)` (인덱스 강제), `IGNORE INDEX (인덱스_이름)` (인덱스 제외) 등 을 사용하여 원하는 인덱스를 사용하도록 통제할 수 있다.

### 그러면 Index를 많이 만들면 좋은건가요?

테이블에 쓰기연산(insert, update, delete)을 실행할 때 마다 index도 변경되므로 index를 적용하게 되면 쓰기 성능이 저하될 수 있다. 또한, 포인터 같은 데이터를 저장하기 때문에 추가적인 저장 공간을 차지한다.

또한, 아래의 경우는 index를 사용하는 것 보다 **full scan**이 더 좋을 수 있다.

1. table에 데이터가 조금 있는 경우 (몇 십, 몇 백건 정도)
2. 조회하려는 데이터가 테이블의 상당 부분을 차지하는 경우 → 찾아보기
3. 데이터가 너무 많아서 index의 크기가 너무 큰 경우

그러므로 무작정 index를 만들기 보다는, 상황을 고려하여 신중히 만들어야 한다.

### Covering index

조회하는 attribute들을 index가 모두 cover할 때를 말한다.

예를 들어, 아래의 쿼리와 같이 id 컬럼으로 조건을 걸었고, 조회하는 데이터도 id라면, index에 있는 데이터로 조회하는 데이터를 모두 cover하게 된다.

```sql
SELECT id, name FROM device WHERE id = 1;
```

이 경우, explain query를 사용했을 때 Extra에 “Using index” 라는 값이 나온다.

![image](https://github.com/JeongHunHui/TIL/assets/108508730/116328a6-ecaf-48a7-b020-0d0f6901347c)

- **장점**
    - 본 테이블까지 가지 않고 index 안에 필요한 모든 데이터가 있기 때문에 조회 성능이 향상된다.
- **단점**
    - 인덱스의 크기가 너무 커진다.

### Hash index

hash table을 사용하여 구현한 index를 말한다.

- **장점**
    - 시간 복잡도가 `O(1)`이다. (매우 빠르다)
- **단점**
    - rehashing에 대한 부담이 있다.
        - hash table은 array로 저장, 데이터가 쌓이다 보면 더 큰 사이즈로 바꿔줘야하는데 이를 rehashing이라고 한다.
    - **동일한지 동일하지 않은지에 대한 비교**만 가능, **범위를 통한 비교**나 **정렬**을 할 때는 **사용이 불가능** 하다.
    - 복합 인덱스의 경우 index의 전체 attribute에 대한 조회만 가능하다.
        - Index(a,b)면 B-Tree 인덱스의 경우는 a에 대한 조회에도 사용 가능하다. 하지만 hash 인덱스는 a와 b에 대한 조회에만 사용 가능하다.
