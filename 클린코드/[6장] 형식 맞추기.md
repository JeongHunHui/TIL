### 형식을 왜 맞춰야 하는가?

같은 형식을 팀 전체가 통일하게 되면 코드의 구조, 형식이 익숙해지면서 코드를 이해하기 쉬워진다.

### 좋은 코드 형식(세로)

1. **적절한 행 길이를 유지하자!**
    
    한 클래스의 줄이 200줄을 넘기지 않도록 하자.
    
    클래스 길이가 길면 길수록 이해가 어려우니 짧게 짜려고 노력하자.
    
2. **개념은 빈 행으로 분리하고, 유사한 개념은 모으자!**
    
    ex
    
    ```java
    public List<Cell> getFlaggedCells() {
        List<Cell> flaggedCells = new ArrayList(); // 변수
    		List<Cell> dummyCells = new ArrayList();
     
        for (Cell cell : gameBoard) // for 문
            if (cell.isFlagged())
                flaggedCells.add(cell);
     
        return flaggedCells; // 반환
    }
    ```
    
3. **코드 순서**
    
    클래스 내에 아래와 같은 순서로 코드를 작성하자.
    
    1. static 변수 : public -> protected -> package -> private
    2. instance 변수 : public -> protected -> package -> private
    3. 생성자
    4. 메서드 : (public -> private -> private) -> (public -> private -> private)...
    - public 메서드에서 호출되는 private 메서드는 바로 아래에 둔다.

### 좋은 코드 형식(가로)

1. **가로 길이를 길게 하지 말자!**
    
    intelij 를 보면 저렇게 선이 있는데, 저기까지 문자가 120글자가 들어간다.
    
    ![image](https://github.com/Google-Developer-Student-Clubs-TUK/2023-01-CleanCode-study/assets/108508730/442a43b6-d230-4bc2-8d83-70ad4a898933)
    
    최대한 120글자는 넘지말자(그렇다고 120자 가까이 채우라는게 아니다.).
    
2. **공백을 잘 넣자!**
    
    `=` `,` `:` 양 옆에 공백을 잘 넣어주면 코드의 가독성이 향상된다.
    
    → 사실 통일하는게 제일 중요하다고 생각합니다(내 의견).
    
3. **정렬을 하지 말아라!**
    
    ex) 아래와 같이 하지 말자.
    
    ```java
    context =                 context;
    socket =                  s;
    requestParsingTimeLimit = 10000;
    ```
    
4. **들여쓰기 무조건 하자!**

### 팀 코딩 컨벤션

구글 java 컨벤션 : [https://google.github.io/styleguide/javaguide.html](https://google.github.io/styleguide/javaguide.html)

네이버 java 컨벤션 : [https://naver.github.io/hackday-conventions-java/](https://naver.github.io/hackday-conventions-java/)

정답은 없다. 위와 같은 코딩 컨벤션과 이 책의 내용을 참고해서 팀원끼리 마음에 드는 코딩 컨벤션을 잘 정하자.

이런 코딩 컨벤션을 매법 신경쓰며 코딩하면 코드는 깔끔할 지라도, 속도가 느려질 수 있다.

→ 각종 코드 포매팅 도구를 활용하자.

참고링크 - inteliJ에서 팀 단위 코드 포매팅 하기

[Github 프로젝트 & Intellij 전반에 걸쳐 Google Java Style Guide 를 강제하기](https://vince-kim.tistory.com/28)

아래와 같이 push / PR 시 github action이 동작하며 포매팅을 해준다.

![image](https://github.com/Google-Developer-Student-Clubs-TUK/2023-01-CleanCode-study/assets/108508730/372219a8-d5d7-43f1-ad7a-32196f1af100)

또한, InteliJ 플러그인으로도 코드 작성 중에 포매팅 적용 가능하다.

JS 같은 경우는 Prettier를 사용하면 된다.
