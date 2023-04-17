# [2장] 의미 있는 이름

### 의도를 분명히 밝혀라

변수, 함수, 클래스 이름은 아래 질문에 모두 답해야함

- 존재 이유는?
- 수행 기능은?
- 사용 방법은?

만약 따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 것

- ex
    
    ```java
    // 이름으로 의도를 드러내지 못한 코드
    int d; // 경과 시간(단위: 날짜)
    
    // 이름으로 의도를 드러낸 코드
    int daysSinceCreation;
    ```
    

아래 함수는 복잡한 로직이 없지만, 함수가 하는 일을 짐작하기 어려움

```java
public List<int[]> getThem() {
	List<int[]> list1 = new ArrayList<int[]>();
	for (int[] x : theList)
		if (x[0] == 4)
			list1.add(x);
	return list1;
}
```

위 코드는 독자가 다음 정보를 안다고 가정

1. theList에 무엇이 들었는가?
2. theList에서 0번째 값이 무엇인가?
3. 값 4는 무슨 의미인가?
4. 함수가 반환하는 리스트 list1을 어떻게 사용하는가?

이러한 정보에 대해 아래처럼 적절하게 이름을 붙혀주기만 해도 코드가 훨씬 나아짐

```java
public List<int[]> getFlaggedCells() {
	List<int[]> flaggedCells = new ArrayList<int[]>();
	for (int[] cell : gameBoard)
		if (cell.isFlagged())
			flaggedCells.add(cell);
	return flaggedCells;
}
```

### 그릇된 정보를 피하라

- 이미 널리 쓰이고 있는 의미가 있는 단어를 다른 의미로 사용하면 안됨
- 여러 계정을 그룹으로 묶을 때 실제 List가 아니면 accountList와 같이 명명 X
- 서로 흡사한 이름을 사용 X
- 유사한 개념은 유사한 표기법 사용

### 의미 있게 구분하라

아래와 같이 같은 범위에 있는 두 변수를 단순히 숫자로 구분하는 것은 좋지 않음

```java
public static void copyChars(char a1[], char a2[]) {
    for (int i = 0; i< a1.length; i++) {
        a2[i] = a1[i];
    }
}
```

아래와 같이 같은 범위의 각 변수에 의미 있는 이름을 붙혀야함(연속된 숫자 → 의미 있는 이름)

```java
public static void copyChars(char source[], char destination[]) {
    for (int i = 0; i< source.length; i++) {
        destination[i] = source[i];
    }
}
```

### 발음하기 쉬운 이름을 사용하라

```java
// 나쁜 코드
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
};

// 좋은 코드
class Customer {
    private Date generationTimeStamp;
    private Date modificationTimeStamp;
    private final String rdcordId = "102";
};
```

### 검색하기 쉬운 이름을 사용하라

의미가 있는 상수나 텍스트는 아래와 같이 눈에 띄고, 검색하기 쉽도록 강조

```java
// 나쁜 코드
if(age >= 20) ...

// 좋은 코드
if(age >= ADULT_AGE) ...
```

### 인코딩을 피하라

1. 헝가리식 표기법(타입을 변수 이름에 쓰는 방식)
    
    자바는 변수 이름에 타입을 인코딩할 필요 X
    
    ```java
    // 나쁜 예시
    PhoneNumber phoneString; // 타입이 바뀌어도 이름은 바뀌지 않는다.
    ```
    
2. 멤버 변수 접두어
    
    멤버 변수에 m_ 이라는 접두어를 붙일 필요 X
    
    ```java
    public class Part {
      private String m_dec; // 설명 문자열
      void setName(PString name) {
          m_dsc = name;
      }
    }
    ```
    
3. 인터페이스 클래스와 구현 클래스
    - `ShapeFactory implements IShapeFactory` 보다
    - `ShapeFactoryImpl implements ShapeFactory`가 낫다.
        
        → 인터페이스 명을 그냥 짓고, 구현 클래스에 Impl을 붙히는 방식
        

### 클래스 이름

- 클래그 이름과 객체 이름은 명사나 명사구가 적합 (동사는 X)
- 좋은 예시: Customer, AddressParser, Account
- Manager, Processor, Date, Info 같은 모호한 단어는 X

### 한 개념에 한 단어만 사용

추상적인 개념 하나에 단어 하나를 선택해 이를 고수

- ex: 정보를 갱신하는 메서드인데 update, renew, change, patch, put  등의 이름 혼용X
- 마찬가지로 동일 코드에 controller, manager, driver 를 섞어쓰면 혼란스러움

→ 일관성 있게 사용하자

### 해법 영역에서 가져온 이름을 사용하라

- 코드를 읽는 사람도 프로그래머이므로 알고리즘 이름, 패턴 이름, 수학 용어 등을 사용
- 기술 개념에는 기술 이름이 가장 적합한 선택

### 문제 영역에서 가져온 이름을 사용하라

- 적절한 프로그래밍 용어가 없다면 문제 영역(도메인 관련)에서 이름을 가져옴
- 문제 영역 개념과 관련이 깊은 코드라면 문제 영역에서 이름을 가져와야함

### 의미 있는 맥락을 추가하라

```java
private void printGuessStatistics(char candidate, int count) {
    String number;
    String verb;
    String pluralModifier;
    if (count == 0) {
        number = "no";
        verb = "are";
        pluralModifier = "s";
    } else if (count == 1) {
        number = "1";
        verb = "is";
        pluralModifier = "";
    } else {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
    String guessMessage = String.format( 
        "There %s %s %s%s", verb, number, candidate, pluralModifier);
    print(guessMessage);
}
```

```java
public class GuessStatisticsMessage {
    private String number;
    private String verb;
    private String pluralModifier;
 
    public String make(char candidate, int count) {
        createPluralDependentMessageParts(count);
        return String.format("There %s %s %s%s", verb, number, candidate, pluralModifier);
    }
 
    private void createPluralDependentMessageParts(int count) {
        if (count == 0) {
            thereAreNoLetters();
        } else if (count == 1) {
            thereIsOneLetter();
        } else {
            thereAreManyLetters(count);
        }
    }
 
    private void thereAreManyLetters(int count) {
        number = Integer.toString(count);
        verb = "are";
        pluralModifier = "s";
    }
 
    private void thereIsOneLetter(){
        number = "1";
        verb = "is";
        pluralModifier = "";
    }
 
    private void thereAreNoLetter(){
        number = "no";
        verb = "are";
        pluralModifier = "s";
    }
}
```

### 불필요한 맥락을 없애라

일반적으로 짧은 이름이 긴 이름보다 좋음. But 의미가 분명해야함

- `accountAddress`와 `customerAddress`는 `Address` 클래스 인스턴스로는 좋은 이름이나 클래스 이름으로는 적합하지 못하다. `Address`는 클래스 이름으로 적합하다.
