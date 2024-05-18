# static의 오해와 진실(feat. k6, VisualVM)

## **0️⃣ 배경 - static 쓰면 안될까?**

코드를 작성할 때 마다 별 생각 없이 static 키워드를 사용했었다. 그럴 때 마다 **static 키워드를 붙히면 메모리를 계속 차지하고 있기 때문에 성능이 저하되므로 자제**하라는 리뷰를 받았다.

왜 메모리를 계속 차지하고 있지? 왜 성능이 저하되지? 라는 의문이 생겨서 이에 대한 궁금증을 해결하기 위해 관련 내용을 공부하고, 직접 테스트한 뒤 결과를 분석한 과정을 정리하였다.

## 1️⃣ JVM 메모리 구조

static 키워드와 메모리에 관련해서 여러 글을 찾다보니, 먼저 JVM의 메모리 구조에 대해 알 필요성을 느끼게 되었다.

### ▶️ Heap과 Metaspace

![image](https://github.com/JeongHunHui/TIL/assets/108508730/07d0d872-b89e-479b-ba83-fa60c845b5de)

`Heap`은 새로 생성된 인스턴스가 저장되는 영역이다. 생성된 인스턴스는 `Heap` 영역에서 돌아다니다가 더 이상 사용되지 않을 때 GC에 의해 메모리가 해제된다.

`Metaspace`는 클래스와 메서드의 메타데이터가 저장되는 영역이고, 네이티브 메모리를 사용한다. Java 8버전 부터 등장했다.

static 변수나 메서드는 두 영역 중 `Metaspace` 영역에 저장된다.

### ▶️ 왜 static 키워드를 붙히면 메모리를 계속 차지하는가?

그 이유는 static 키워드를 붙힌 변수나 메서드는 속한 클래스와 같은 생명주기를 가지기 때문이다. 즉, 속한 클래스가 로드될 때 `Metaspace` 영역에 저장되고, 언로드될 때 GC의 대상이 되어 메모리가 해제된다는 의미이다.

하지만, JVM에서 클래스를 언로드하는 경우는 매우 드물기 때문에 계속 메모리를 차지하고 있는 것이다.

그렇다면 static 변수에 많은 데이터를 넣으면 실제로 어떻게 될까?

## 2️⃣ static List에 인스턴스 삽입 테스트

이론적으로는 예상이 되지만, 과연 static 변수에 계속 데이터를 넣으면 어떻게 되는지 직접 테스트 하고 싶어졌다.

### ▶️ 테스트 과정

1. static List를 선언하고, 해당 List에 값을 넣는 API를 만든다.
2. k6를 활용한 부하 테스트로 static List에 많은 데이터를 넣고 API 통신 지표를 측정한다.
3. VisualVM을 활용하여 해당 과정에서 JVM을 모니터링 하여 여러 지표를 측정한다.
4. 다양한 지표(**Heap Dump**, CPU 및 메모리 사용량 그래프 등)를 통해서 결과를 분석한다.

### ▶️ 테스트 환경

- Kotlin + SpringBoot 환경
- JVM 메모리 최대 사용량을 `256mb`로 제한(`-Xmx256m`)
    - 메모리 용량을 낮춰 메모리 사용에 더 민감하게 하기 위함
- k6의 가상 유저 수는 `2500`, 테스트 시간은 `10초`로 설정
    - 테스트 시간을 짧게 설정하고 여러 번 수행하여 평균 계산

### ▶️ 테스트 할 코드

- github 주소: https://github.com/JeongHunHui/StaticKeywordTest

- `FruitClass` - 과일의 정보를 담고 있는 객체
    
    ```kotlin
    data class FruitClass (
        val koreanName: String,
        val price: Int,
        val description: String
    )
    ```
    
- `StaticTestController` - FruitClass 인스턴스를 저장하는 API를 구현한 컨트롤러
    
    ```kotlin
    @RestController
    class StaticTestController {
        companion object {
            val fruitClasses = arrayListOf<FruitClass>()
        }
    
        @GetMapping("/nonStaticFruitClass")
        fun saveNonStaticFruitClass() {
            for (i in 0..2) {
                fruitClasses.add(FruitClass("사과", 1000, "붉고 맛있는 과일"))
                fruitClasses.add(FruitClass("바나나", 1500, "길고 노란 열대 과일"))
                fruitClasses.add(FruitClass("체리", 2000, "작고 달콤한 빨간 과일"))
                fruitClasses.add(FruitClass("포도", 3000, "다수의 작은 과일이 한송이에 달려있는 과일"))
                fruitClasses.add(FruitClass("오렌지", 2500, "비타민 C가 풍부한 주황색 과일"))
            }
        }
    }
    ```
    

### ▶️ 테스트 결과

- K6 부하 테스트 결과(스크린샷)

  ![image](https://github.com/JeongHunHui/TIL/assets/108508730/cc54df64-9030-42be-a59b-69aeffdebdc7)
    
- K6 부하 테스트 결과(표)
    
    
    | 지표 | 1차 테스트 | 2차 테스트 | 3차 테스트 | 평균 |
    | --- | --- | --- | --- | --- |
    | 데이터 수신량 (MB) | 21 | 21 | 21 | 21 |
    | 그룹 지속 시간 (ms) | 86.97 | 97.85 | 90.48 | 91.77 |
    | HTTP 요청 응답 시간 (ms) | 53.44 | 69.95 | 58.9 | 60.76 |
    | 반복 수행 시간 (ms) | 90.85 | 102.01 | 94.33 | 95.73 |
    | 반복 수행 횟수 | 287629 | 286628 | 286133 | 286797 |
- VisualVM 모니터링 결과
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/3cb776a9-44af-4d34-be66-8631912171b3)
    

### ▶️ 테스트 결과 분석 - OOM 발생

![image](https://github.com/JeongHunHui/TIL/assets/108508730/0f1ba83b-2efc-4ec9-aee2-57005e136f39)

우선, 테스트를 진행하며 위와 같이 OOM(Out Of Memory)에러가 발생하였다. 이에 대한 원인은 VisualVM을 통해 파악할 수 있었다.

![image](https://github.com/JeongHunHui/TIL/assets/108508730/639f7ef0-3e46-4e95-850a-b7a152ab45ac)

Heap 메모리 사용량 그래프를 보면, 메모리 사용량이 줄어들지 않고 우상향하고 있다. GC가 작동하면서 메모리를 확보해야하는데 그렇지 못하고 있다. 이는 전형적인 메모리 누수 현상이다.

그렇다면 메모리는 어떤 데이터를 가지고 있을까? 이는 **Heap Dump**를 통해 확인할 수 있다.

![image](https://github.com/JeongHunHui/TIL/assets/108508730/3f6a4d9b-172f-42a9-bdac-b9231b63bd1b)

**Heap Dump**를 확인해보니, `FruitClass`가 전체 인스턴스의 `91.8%`인 `4121115개` 생성되었고, 용량도 전체의 `63.1%`에 해당하는 약 `95MB`를 차지하고 있는 모습이다.

정리하면 static List에 `FruitClass`가 계속 추가됐고, GC가 메모리를 확보하지 못해 Heap 메모리가 `FruitClass`로 가득 차며 OOM이 발생하였다.

그렇다면 왜 GC가 메모리를 확보하지 못했을까?

static List는 Metaspace 영역에 저장되어있고, 속해있는 클래스(`StaticTestController`)와 생명주기를 함께한다. GC는 참조가 없는 인스턴스들을 수거하는데, static List에 저장된 FruitClass 인스턴스들은 계속 static List에 의해 참조되고 있기 때문에 GC가 메모리를 확보하지 못했다.

### ▶️ 테스트 결론

static 키워드가 참조하는 인스턴스들은 계속 Heap 메모리를 차지하고 있으므로 사용을 주의해야한다. 또한, 이런 메모리 누수(Memory Leak)현상을 확인하고 원인을 분석하는 과정에 대해서도 배우게 되었다.

## 3️⃣ static 키워드는 쓰면 안되는걸까?

사실 static 키워드가 붙은 상수, 클래스, 메서드 자체는 Heap 영역이 아닌 Metaspace 영역에 저장되고, 그 자체로 많은 메모리를 차지하기는 어렵다. 대신, 위 테스트 처럼 **static 변수가 참조하는 인스턴스가 매우 많거나, 엄청 큰 데이터를 저장하는 방식으로 사용해서는 안된다.**

### ▶️ static 키워드는 언제 써야할까?

위 테스트에서는 아래와 같이 반복되는 값들에 대해서 일일히 인스턴스를 생성해서 List에 삽입하였다.

```kotlin
@RestController
class StaticTestController {
    companion object {
        val fruitClasses = arrayListOf<FruitClass>()
    }

    @GetMapping("/nonStaticFruitClass")
    fun saveNonStaticFruitClass() {
        for (i in 0..2) {
            fruitClasses.add(FruitClass("사과", 1000, "붉고 맛있는 과일"))
            fruitClasses.add(FruitClass("바나나", 1500, "길고 노란 열대 과일"))
            fruitClasses.add(FruitClass("체리", 2000, "작고 달콤한 빨간 과일"))
            fruitClasses.add(FruitClass("포도", 3000, "다수의 작은 과일이 한송이에 달려있는 과일"))
            fruitClasses.add(FruitClass("오렌지", 2500, "비타민 C가 풍부한 주황색 과일"))
        }
    }
}
```

하지만, 이렇게 반복되는 값에 아래 코드 처럼 static 키워드를 붙혀서 저장한다면 일일히 인스턴스를 생성하는 것 보다 효율적으로 메모리를 사용할 수 있게 된다.

```kotlin
data class FruitClass (
    val koreanName: String,
    val price: Int,
    val description: String
) {
    companion object {
        val APPLE = FruitClass("사과", 1000, "붉고 맛있는 과일")
        val BANANA = FruitClass("바나나", 1500, "길고 노란 열대 과일")
        val CHERRY = FruitClass("체리", 2000, "작고 달콤한 빨간 과일")
        val GRAPE = FruitClass("포도", 3000, "다수의 작은 과일이 한송이에 달려있는 과일")
        val ORANGE = FruitClass("오렌지", 2500, "비타민 C가 풍부한 주황색 과일")
    }
}
```

과연 정말 그럴지 테스트를 진행해서 결과를 비교해보자.

## 4️⃣ static List에 static 값 삽입 테스트

### ▶️ 테스트 과정, 환경, 코드

테스트 과정 및 환경은 이전 테스트와 동일하고, 코드는 아래와 같이 변경하였다.

- `FruitClass`
    
    ```kotlin
    data class FruitClass (
        val koreanName: String,
        val price: Int,
        val description: String
    ) {
        companion object {
            val APPLE = FruitClass("사과", 1000, "붉고 맛있는 과일")
            val BANANA = FruitClass("바나나", 1500, "길고 노란 열대 과일")
            val CHERRY = FruitClass("체리", 2000, "작고 달콤한 빨간 과일")
            val GRAPE = FruitClass("포도", 3000, "다수의 작은 과일이 한송이에 달려있는 과일")
            val ORANGE = FruitClass("오렌지", 2500, "비타민 C가 풍부한 주황색 과일")
        }
    }
    ```
    
- `StaticTestController`
    
    ```kotlin
    @RestController
    class StaticTestController {
        companion object {
            val fruitClasses = arrayListOf<FruitClass>()
        }
    
        @GetMapping("/fruitClass")
        fun saveFruitClass() {
            for (i in 0..2) {
                fruitClasses.add(FruitClass.APPLE)
                fruitClasses.add(FruitClass.BANANA)
                fruitClasses.add(FruitClass.CHERRY)
                fruitClasses.add(FruitClass.GRAPE)
                fruitClasses.add(FruitClass.ORANGE)
            }
        }
    }
    ```
    

### ▶️ 테스트 결과

- K6 부하 테스트 결과(스크린샷)
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/aae4c401-3b65-4795-811e-e7106204b837)
    
- K6 부하 테스트 결과(표)
    
    
    | 지표 | 1차 테스트 | 2차 테스트 | 3차 테스트 | 평균 |
    | --- | --- | --- | --- | --- |
    | 데이터 수신량 (MB) | 26 | 26 | 25 | 25.67 |
    | 그룹 지속 시간 (ms) | 59.81 | 63.52 | 63.2 | 62.18 |
    | HTTP 요청 응답 시간 (ms) | 31.17 | 37.41 | 35.59 | 34.72 |
    | 반복 수행 시간 (ms) | 64.13 | 66.8 | 67.78 | 66.24 |
    | 반복 수행 횟수 | 353542 | 350561 | 338023 | 347375 |
- VisualVM 모니터링 결과
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/16c2c897-8a70-46c2-a5cc-8988e43c77fa)
    

### ▶️ 테스트 결과 분석

우선 1차 테스트에서 발생했던 OOM 에러는 발생하지 않았다.

그리고 Heap Dump를 확인해보면, 1차 테스트에서는 `FruitClass`가 Heap 영역의 대부분을 차지했었지만, 2차 테스트에서는 없다는 것을 확인할 수 있다.

- 1차 테스트 Heap Dump (인스턴스)
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/b967d902-ee72-4905-8b6a-a71c8b9d1e6d)
    
- 2차 테스트 Heap Dump (static)
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/91ffe188-df4d-4acd-a723-a5d26bbe5cf8)
    

또한, 1차 테스트와 2차 테스트의 K6 지표를 비교해보면 더 많은 데이터를 수신하고, 짧은 지속시간과 응답 시간이 소요되고, 더 많은 처리를 했다는 점을 통해 성능적으로 우수하다는 점을 확인할 수 있다.

- 1차 테스트 K6 지표 (인스턴스)
    
    
    | 지표 | 1차 테스트 | 2차 테스트 | 3차 테스트 | 평균 |
    | --- | --- | --- | --- | --- |
    | 데이터 수신량 (MB) | 21 | 21 | 21 | 21 |
    | 그룹 지속 시간 (ms) | 86.97 | 97.85 | 90.48 | 91.77 |
    | HTTP 요청 응답 시간 (ms) | 53.44 | 69.95 | 58.9 | 60.76 |
    | 반복 수행 시간 (ms) | 90.85 | 102.01 | 94.33 | 95.73 |
    | 반복 수행 횟수 | 287629 | 286628 | 286133 | 286797 |
- 2차 테스트 K6 지표 (static)
    
    
    | 지표 | 1차 테스트 | 2차 테스트 | 3차 테스트 | 평균 |
    | --- | --- | --- | --- | --- |
    | 데이터 수신량 (MB) | 26 | 26 | 25 | 25.67 |
    | 그룹 지속 시간 (ms) | 59.81 | 63.52 | 63.2 | 62.18 |
    | HTTP 요청 응답 시간 (ms) | 31.17 | 37.41 | 35.59 | 34.72 |
    | 반복 수행 시간 (ms) | 64.13 | 66.8 | 67.78 | 66.24 |
    | 반복 수행 횟수 | 353542 | 350561 | 338023 | 347375 |

그렇다면 왜 성능적으로 우수한걸까?

### ▶️ 두 테스트 간 성능 차이의 원인

성능 테스트에서 static 키워드를 사용하는 것이 인스턴스를 사용하는 것보다 더 효율적인 결과를 보인 이유는 크게 2가지가 있다.

1. **객체 생성 및 메모리 할당 작업이 없다.**
    
    위에서 설명했듯이 static 키워드를 붙힌 요소는 프로그램 실행 시 메모리에 할당되고, 프로그램의 생명주기 동안 계속 유지된다. 이 때문에 매번 새로운 인스턴스를 생성하고 메모리에 할당할 필요가 없으므로 성능이 향상된다.
    
2. **GC가 비효율적으로 동작한다.**
    
    두 테스트에서 GC가 일어난 횟수와 GC 작업에 소요된 총 시간을 비교해보면, 횟수는 3배 더 많았고, 소요 시간은 22배 더 많았다.
    
    - 1차 테스트 GC 통계 (인스턴스)
        
        ![image](https://github.com/JeongHunHui/TIL/assets/108508730/40fd2613-de9e-45f4-aab6-2cb1611fbf3c)
        
    - 2차 테스트 GC 통계 (static)
        
        ![image](https://github.com/JeongHunHui/TIL/assets/108508730/f13809c9-8895-45c5-b5b1-96f41a2fc2d3)
        
    
    당연히 Heap 메모리를 계속 차지하게 되면, GC는 메모리를 확보하기 위해 동작하게 된다. 하지만 대부분 참조가 일어나고 있기 때문에 GC가 메모리 해제를 시킬 수 없으며, 계속 메모리를 차지하고 있기 때문에 수거하기 위해 스캔할 데이터도 더 많다.
    
    GC는 작동하게되면 어플리케이션 실행을 멈추는 `Stop The World`를 발생시키기 때문에 더 자주, 그리고 더 길게 실행하게 되면 당연히 성능에 악영향을 미치게된다.
    

## 5️⃣ 결론 - 잘 쓰면 좋다!

static 키워드를 붙히면 메모리를 계속 차지한다. 그러므로 static 키워드를 붙힌 데이터의 용량이 크거나, 참조하는 값이 많다면 성능에 악영향을 끼치는 것은 물론이고, 메모리 누수로 인한 OOM까지 발생할 수 있다.

하지만 자주 사용되는 상수나 메서드에 static 키워드를 붙히면 더 효과적으로 메모리를 사용하여 성능에도 좋은 영향을 미치게된다.

그러므로 static 키워드는 적절한 상황에 잘 사용하는 것이 중요하다.

또한, 위 과정에서 **JVM의 메모리 구조**, **K6를 이용한 부하 테스트**, **VisualVM을 이용한 JVM 모니터링**까지 많은 내용을 학습할 수 있었다.

다음에는 내 프로젝트에 적용시켜서 내 프로젝트에서는 메모리 누수같은 문제가 일어날 지 테스트 해봐야겠다.

## 6️⃣ 부록 - Enum은 어떨까?

문득 위 테스트를 Enum으로 하면 다른 결과가 나올지 궁금해 졌다. 그래서 진행해봤다.

- 코드
    - `FruitEnum`
        
        ```kotlin
        enum class FruitEnum(
            private val koreanName: String,
            private val price: Int,
            private val description: String
        ) {
            APPLE("사과", 1000, "붉고 맛있는 과일"),
            BANANA("바나나", 1500, "길고 노란 열대 과일"),
            CHERRY("체리", 2000, "작고 달콤한 빨간 과일"),
            GRAPE("포도", 3000, "다수의 작은 과일이 한송이에 달려있는 과일"),
            ORANGE("오렌지", 2500, "비타민 C가 풍부한 주황색 과일")
            ;
        }
        ```
        
    - `StaticTestController`
        
        ```kotlin
        @RestController
        class StaticTestController {
            companion object {
                val fruitEnums = arrayListOf<FruitEnum>()
            }
        
            @GetMapping("/fruitEnum")
            fun saveFruitEnum() {
                for (i in 0..2) {
                    fruitEnums.add(FruitEnum.APPLE)
                    fruitEnums.add(FruitEnum.BANANA)
                    fruitEnums.add(FruitEnum.CHERRY)
                    fruitEnums.add(FruitEnum.GRAPE)
                    fruitEnums.add(FruitEnum.ORANGE)
                }
            }
        }
        ```
        
- K6 부하 테스트 결과(스크린샷)
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/a4595679-3312-49a0-9d35-5f701b26abda)
    
- VisualVM 모니터링 결과
    
    ![image](https://github.com/JeongHunHui/TIL/assets/108508730/9888a97f-e897-4c76-8241-5aee3480092c)
    

테스트 해보니 큰 차이는 없었다 😅
