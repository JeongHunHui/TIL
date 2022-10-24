# Redis란?

### 참조

- Redis란?
    
    [https://devlog-wjdrbs96.tistory.com/374](https://devlog-wjdrbs96.tistory.com/374)
    
- 레디스 튜토리얼
    
    [https://meetup.toast.com/posts/224](https://meetup.toast.com/posts/224)
    

### Redis란?

> **RE**mote **DI**ctionary **S**erver, 고성능 key-value 저장소로서 문자열, 리스트, 해시, Sorted-Set 등 다양한 형식의 데이터를 지원하는 NoSQL, 인메모리 데이터베이스이다.
> 

### Redis의 특징

1. 레디스는 In-Memory 데이터베이스이다.
    
    즉, 모든 데이터를 메모리에 저장하고 조회
    
    - 메모리 접근이 디스크 접근보다 빠름
    - → 기존 관계형 데이터베이스 보다 훨씬 빠름
2. 다양한 자료구조
    
    ![image](https://user-images.githubusercontent.com/108508730/197576482-09e808b9-9903-468a-947c-f7e96fb73e63.png)
    
    레디스는 위와 같이 다양한 자료구조를 Key-Value 형태로 저장한다.
    
    → 개발의 편의성이 높아지고, 난이도가 낮아진다.
    

### Redis 자료구조

1. string
    
    ![image](https://user-images.githubusercontent.com/108508730/197576537-1c7f433c-38d0-4017-84b3-aa98047d4d61.png)
    
    redis의 키가 문자열이므로 문자열을 다른 문자열에 매핑하는 것이라고 볼 수 있다.
    
    string 타입에는 모든 종류의 문자열(이진 데이터 포함)을 저장할 수 있다.
    
2. list
    
    ![image](https://user-images.githubusercontent.com/108508730/197576640-d9877592-3f6c-4edf-9a18-42d1889e9399.png)
    
    레디스의 list는 linked list의 특징을 갖고 있다. → list 의 아이템수와 상관없이 head와 tail에 값을 추가할 떄 동일한 시간이 소요된다. 특정 값이나 인덱스로 데이터를 찾거나 삭제할 수 있다.
    
3. HASH
    
    ![image](https://user-images.githubusercontent.com/108508730/197576715-7a5aad28-ced5-4623-bd00-86d774848931.png)
    
    hash는 field-value 쌍을 사용한 일반적인 해시다. RDB의 table과 비슷한 느낌이다.
    
4. set
    
    ![image](https://user-images.githubusercontent.com/108508730/197576764-480fc953-f028-464c-9c23-e9269cc5377b.png)
    
    set은 정렬되지 않은 문자열의 모음이다. 집합 연산을 수행할 수 있기 때문에 객체 간의 관계를 표현할 때 좋다.
    
5. sorted set
    
    ![image](https://user-images.githubusercontent.com/108508730/197576806-f93a162a-7576-42a2-b2da-2fcd34efe9cc.png)
    
    sorted set은 set과 마찬가지로 key하나에 중복되지 않는 여러 멤버를 저장하지만, 각각의 멤버는 스코어에 연결된다. 모든 데이터는 score 값으로 정렬되며 주로 sort가 필요한 곳에 사용된다.
    
    sorted set은 정렬된 형태로 저장되므로 인덱스를 이용하여 빠르게 조회할 수 있다.
    
    → 인덱스를 이용하여 조회할 일이 많다면 list보다는 sorted set이 좋음
    
6. 기타 자료구조
    - bit / bitmap
    - hyperloglogs
    - Geospatial indexes
    - Stream
    

### Redis Key

레디스의 키는 문자열이며 빈 문자열도 키가 될 수 있다.

키를 조회할 때의 비용을 생각하면, 키를 너무 길게 사용하는 것은 좋지 않다.

레디스의 키를 잘 설계하는 것도 중요하다. 어떻게 생성하느냐에 따라 분산이 몰릴수도 있다. 보통 스키마를 사용해서 레디스의 키를 설계하는 것이 좋은데, `object-type:id` 의 형태를 권장한다.

### Expire 기능

레디스는 인메모리 DB인 만큼 메모리에 저장하는 데이터는 한정적이다. 더이상 메모리에 데이터를 저장할 수 없는 경우 레디스에서는 가장 먼저 들어온 데이터를 삭제하거나, 가장 최근에 사용되지 않은 데이터를 삭제한다.

→ 삭제되는 데이터를 레디스에게 맡기기 보다는 직접 데이터를 입력할 때 데이터의 사용 기한이 언제까지인지 설정해 주는 식으로 설정하는 것이 좋다.
