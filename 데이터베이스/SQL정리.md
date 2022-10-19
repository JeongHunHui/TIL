### create table 문

```sql
create table 테이블명(
	속성명 타입 제약조건(선택),
	...
	// 키의 정의(pk, fk, unique, check)(선택)
);
```

**제약조건**

- not null
- default 초기값

**키의 정의**

- primary key(속성명)
    - 기본키를 지정하는 키워드, null 값 불가
- unique(속성명)
    - 대체키를 지정하는 키워드, null 허용, 다른 값과 중복 X
- foreign key(속성명) references 테이블명(해당 테이블 PK)
    - 왜래키를 지정하는 키워드
    - 참조 무결성 제약조건 유지를 위해 참조되는 테이블에서 투플 삭제 시 처리 방법을 지정하는 옵션
        - ON DELETE NO ACTION : 투플을 삭제하지 못하게 함
        - ON DELETE CASCADE: 관련 투플을 함께 삭제함
        - ON DELETE SET NULL: 관련 투플의 왜래키 값을 NULL로 변경
        - ON DELETE SET DEFAULT: 관련 투플의 왜리키 값을 미리 지정한 기본 값으로 변경
    - 참조 무결성 제약조건 유지를 위해 참조되는 테이블에서 투플 변경 시 처리 방법을 지정하는 옵션
        - ON UPDATE NO ACTION: 투플을 변경하지 못하게 함
        - ON UPDATE CASCADE: 관련 투플에서 왜래키 값을 함께 변경함
        - ON UPDATE SET NULL: 관련 투플의 왜래키 값을 NULL로 변경함
        - ON UPDATE SET DEFAULT: 관련 투플의 왜리키 값을 미리 지정한 기본 값으로 변경
- check(조건문)
    - 데이터 무결성 제약조건을 지정해줌
    - CONSTRAINT CHK_CPY CHECK(제조업체 = ‘한빛제과’) 이런식으로 이름 부여 가능

---

### alter table 문

새로운 속성 추가

```sql
alter table 테이블이름
add 속성명 타입 제약조건(선택);
```

기존 속성 삭제

```sql
alter table 테이블이름 drop column 속성명;
```

새로운 제약조건 추가

```sql
alter table 테이블이름 add constraint 제약조건이름 제약조건내용;
```

기존 제약조건의 삭제

```sql
alter table 테이블이름 drop constraint 제약조건이름;
```

---

### drop table 문

```sql
drop table 테이블명;
```

---

# 데이터 검색

### select 문

```sql
select 속성명1, 속성명2, ...
from 대상테이블1, 대상테이블2, ...
where 조건문;
```

모든 속성 검색 → * 사용

```sql
select * from 고객;
```

→ 고객테이블의 모든 속성 검색

**all:** 중복되는 값 까지 전부 검색(기본값)

![image](https://user-images.githubusercontent.com/108508730/196723777-a9f594bd-2346-47b4-a8c8-071e092b3bf3.png)


**distinct:** 중복되는 값 제거

![image](https://user-images.githubusercontent.com/108508730/196723843-2e71b188-5fa0-4a96-8041-af07121ec03e.png)

**as**: 별명 지정 가능

![image](https://user-images.githubusercontent.com/108508730/196723945-661c48fe-65e2-42d3-8bd1-78ab90f44fb6.png)

- 산술식을 이용한 검색
    
    ![image](https://user-images.githubusercontent.com/108508730/196724005-0433a1d0-7380-4ad0-960e-878747bac8ff.png)
    

---

### 조건 검색

[where 조건]

비교연산자 <>: 다르다

예시

![image](https://user-images.githubusercontent.com/108508730/196724134-c8240825-957b-4764-994e-250c94f96941.png)

**like 키워드**

- 부분적으로 일치하는 데이터 검색
- 문자열을 이용하는 조건에만 like키워드 사용 가능
- 예시
    
    ```sql
    ...
    where 고객이름 like '_한%';
    ```
    
    → 고객이름의 두번째 글자가 ‘한’인 데이터 검색
    

**is null, is not null 키워드**

해당 데이터가 null 이면 참

**order by 키워드**

order by 속성1 [asc | desc] 속성2(선택) [asc | desc]

→ 해당 속성으로 오름차순(asc), 내림차순(desc) 정렬

→ 속성이 여러개인 경우 속성1로 정렬하고 속성1이 같은 경우 속성2로 정렬

**집계함수**

- null값인 경우 count하지 않음

**그룹별 검색**

group by 키워드를 이용해 특정 속성의 값이 같은 투플을 모아 그룹을 만들고, 그룹별로 검색

having 키워드와 함께 그룹에 대한 조건 작성 가능

```sql
[where 조건]
[group by 속성리스트 [having 조건]]
```

예시1 - 주문 테이블에서 주문제품별 수량의 합계를 검색하시오

```sql
select 주문제품, sum(수량) as 총주문수량
from 주문
group by 주문제품;
```

→ 동일 제품을 주문한 투플을 모아 그룹으로 만들고 그룹별로 수량의 합계를 계산

예시2 - 제품 테이블에서 제품을 3개 이상 제조한 제조업체별로 제조한 제품의 개수와 제품중 가장 비싼 단가를 검색하시오.

```sql
select 제조업체, count(*) as 제품수, max(단가) as 최고가
from 제품
group by 제조업체 having count(*) >= 3;
```

→ 제품 개수가 3개이상인 제조업체별로 제품의 개수와 제조업체별 제품의 최고가를 검색하였다.

### 조인 검색

예시1 - banana 고객이 주문한 제품의 이름을 검색하시오

```sql
select 제품명
from 제품, 주문
where 주문고객 = 'banana' and 제품번호 = 주문제품;
```

예시2 - 나이가 30이상인 고객이 주문한 제품의 번호와 주문일자를 검색하시오

```sql
select 주문제품, 주문일자
from 주문, 고객
where 주문고객 = 고객아이디 and 나이 >= 30;
```

예시3 - 고명석 고객이 주문한 제품의 제품명을 검색하시오

```sql
select 제품명
from 주문, 고객, 제품
where 고객이름 = '고명석' and 주문제품 = 제품번호 and 고객아이디 = 주문고객;
```

참고 - 조인 정리

![image](https://user-images.githubusercontent.com/108508730/196724240-4e45e1ea-73cb-4fea-80c6-f850c9cdfd82.png)

**내부조인**

from 테이블1 inner join 테이블2 on 조인조건

**외부조인**

from 테이블1 left | right | full outer join 테이블2 on 조인조건

### 부속 질의문

예시1 - 달콤비스킷을 생상한 제조업체가 만든 제품들의 제품명과 단가를 검색하시오

```sql
select 제품명, 단가
from 제품
where 제조업체 = (select 제조업체
								from 제품
								where 제품명 = '달콤비스킷');
```

부속 질의문 연산자

예시2 - banana 고객이 주문한 제품의 제품명과 제조업체를 검색하시오

```sql
select 제품명, 제조업체
from 제품
where 제품번호 in (select 주문제품
                    from 주문
                    where 주문고객 = 'banana')
```

→ banana고객이 주문한 제품들 중에서 제품번호가 일치하는 제품의 제품명과 제조업체 출력

예시3 - 대한식품이 제조한 모든 제품의 단가보다 비싼 제품의 제품명, 단가, 제조업체를 검색하시오

```sql
select 제품명, 단가, 제조업체
from 제품
where 단가 > all (select 단가
                    from 제품
                    where 제조업체 = '대한식품')
```

→ 제조업체가 대한식품인 제품의 단가 전부보다 단가가 높은 제품의 제품명, 단가, 제조업체 출력

---

### insert 문

```sql
insert
into 테이블명[(속성리스트)]
values (속성값리스트);
```

예시1 - 고객테이블에 고객아이디가 strawbarry, 고객이름이 최유경, 나이가 30, 등급이 vip, 직업이 공무원, 적립금이 100인 고객의 정보를 삽입 후 확인해보시오

```sql
insert into 고객(고객아이디, 고객이름, 나이, 등급, 직업, 적립금)
values('strawberry', '최유경', 30, 'vip', '공무원', 100);
select * from 고객;
```

**insert + select문**

```sql
insert into 테이블이름[(속성리스트)]
select 문;
```

→ select문으로 검색한 데이터를 insert

---

### update 문

```sql
update 테이블_이름
set 속성이름1 = 값1, 속성이름2 = 값2, ...
[where 조건];
```

예시1 - 정소화 고객이 주문한 제품의 주문수량을 5개로 수정하시오

```sql
update 주문
set 수량 = 5
where 주문고객 = (select 고객아이디
                    from 고객
                    where 고객이름 = '정소화');
```

→ 고객이름이 정소화인 고객의 고객아이디와 주문고객이 같은 주문의 수량을 5로 수정

---

### delete 문
```sql
delete from 테이블이름
[where 조건];
```
