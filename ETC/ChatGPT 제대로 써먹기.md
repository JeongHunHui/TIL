### 서론

이 글을 작성하게 된 계기는.. 1달 정도 ChatGPT를 활발하게 사용하며, 개발 생산성이 정말 많이 향상되었고, 이 과정에서 얻은 팁과 효과적인 활용방법을 공유하기 위해서입니다.

구글링과 비교하면, 구글링으로는 찾기 번거로운 나의 상황에 최적화된 정보를 얻을 수 있습니다.

ex: SpringBoot 3.0.0 환경에서 이러한 코드를 사용하였는데 다음과 같은 예외가 발생했다. 어떻게 해결할 수 있는가?

→ 구체적인 사용 예시는 뒤에서 다루겠습니다.

![image](https://user-images.githubusercontent.com/108508730/224327717-89fcc8fc-eb80-435e-95e4-35b1b9d7dadd.png)

### ChatGPT란?

ChatGPT란 OpenAI에서 개발한 대형 언어 모델입니다. 트랜스포머라고 불리는 신경망 아키텍처를 사용하여 방대한 양의 텍스트 데이터에 대한 교육을 받아서 자연어 입력에 대해 인간과 같은 반응을 생성하도록 설계되었습니다. ChatGPT는 텍스트 완성, 번역, 요약, 대화형 AI 등 다양한 작업에 활용될 수 있습니다.

![사실 이 내용도 ChatGPT에서 가져왔습니다..](https://user-images.githubusercontent.com/108508730/224327777-fb26ede3-10cd-4547-a089-789d47b95f2b.png)

사실 이 내용도 ChatGPT에서 가져왔습니다..

### ChatGPT의 장단점

**장점**

- 내 상황을 구체적으로 설명한다면 구글링에 비해 **나의 상황에 맞는 최적화된 답변**을 받을 수 있다.
- 아래의 몇 가지 단점들을 크롬 확장프로그램으로 보완할 수 있다.

**단점**

- 2021년까지의 데이터를 학습하였기 때문에 최신 정보가 필요한 경우 답변의 정확성이 떨어진다.
- 한글로 질문을 하였을 때 속도가 매우 저하된다.
- 가끔 접속자가 몰리는 시간대에는 작동이 원활하지 않은 경우가 있다.

### ChatGPT와 함께 사용할 때 유용한 크롬 확장 프로그램들 & 이를 활용한 구체적인 사용 예시

1. 프롬프트 지니: ChatGPT 자동 번역기
    
    ![image](https://user-images.githubusercontent.com/108508730/224327928-8feb1fdb-7a68-481b-ba5a-2748e58d0bbe.png)
    
    https://chrome.google.com/webstore/detail/프롬프트-지니-chatgpt-자동-번역기/lhkgpdljnlplgbkonflbhifackjhjmdj?hl=ko
    
    ChatGPT 내에서도 한글로 질문이 가능하지만, 속도가 매우 느립니다. 그래서 보통 외부 번역기를 사용하여 영어로 질문하고, 나온 답변을 다시 번역하는 방식으로 이용하는 경우가 있습니다.
    
    그럴 때 이 확장 프로그램을 사용하면, 영어로 질문했을 때와 비슷한 속도로 한글 답변을 받을 수 있습니다.
    
    사용 에시
    
    ![image](https://user-images.githubusercontent.com/108508730/224327974-a0969304-3c5d-4501-9cfe-dca87b2b08a3.png)
    
    ![image](https://user-images.githubusercontent.com/108508730/224328536-f5444261-d967-438a-aee4-2f938d54782f.png)
    

1. AIPRM for ChatGPT
    
    ![image](https://user-images.githubusercontent.com/108508730/224329003-3f63f7b5-e6e5-496f-8b97-fcd230fb6399.png)
    
    [https://chrome.google.com/webstore/detail/aiprm-for-chatgpt/ojnbohmppadfgpejeebfnmnknjdlckgj?hl=ko](https://chrome.google.com/webstore/detail/aiprm-for-chatgpt/ojnbohmppadfgpejeebfnmnknjdlckgj?hl=ko)
    
    ![image](https://user-images.githubusercontent.com/108508730/224329221-0dac874f-0832-46e6-b651-a3022b852d28.png)
    
    ChatGPT에게 질문을 할 때 어떤 식으로 해야할지 헷갈리는 경우가 많은데, 여러 종류의 질문들을 템플릿 형식으로 만들어서 편하게 사용할 수 있도록 한 확장프로그램 입니다.
    
    예를 들면, 파이썬으로 이러한 코드를 만들고 싶다는 질문을 한다면, 원래는 `파이썬을 이용하여 ~한 코드를 만들어달라` 해야겠지만, 여기서 `~` 만 입력하면 되도록 하는 템플릿을 쉽게 적용시킬 수 있습니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329324-f3ddb4f8-3bac-4e22-95d5-19ad1333c10b.png)
    
    프로그래머스 2단계 문제 “피보나치 수”
    
    만약 이 알고리즘 문제를 푼다고 하면 AIPRM의 `Software Engineering` Topic의 `Python Pro` 를 활용해볼 수 있습니다.
    
    해당 문제를 설명하는 이 부분을 PythonPro 템플릿을 적용시켜서 질문해보겠습니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329382-2594c322-0210-42e7-9891-ac8297e7aa19.png)
    
    ![image](https://user-images.githubusercontent.com/108508730/224329433-c13a7ee5-1567-4ac1-8429-41f1792087cd.png)
    
    아래와 같은 결과가 나왔습니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329506-3996a7d9-dc2e-44dd-bdf1-55287a0eb980.png)
    
    이 코드를 프로그래머스에 그대로 입력하니, 테스트를 전부 통과했습니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329558-523af56c-1cc1-40d6-a158-36ab8eda88d9.png)
    
    꼭 개발이 아니더라도 다른 분야에서도 다양한 활용방법이 있습니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329645-5662d732-2461-4145-a0c3-a30dcdc8f443.png)
    
    예를 들어, 특정 웹사이트를 요약하고 싶다면, AIPRM의 `Productivity` Topic의 `Analyze the Website Summarize` 를 활용해볼 수 있습니다.
    
    템플릿을 적용시킨 뒤 Redis의 위키백과 URL을 넣으니, 아래와 같은 답변을 받았습니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329722-e6dcdc01-dce8-4f81-9b6b-9f43c3d768ad.png)
    
    ![image](https://user-images.githubusercontent.com/108508730/224329759-34845346-694f-4ff7-b06a-87333f004bfd.png)
    
2. WebChatGPT
    
    ![image](https://user-images.githubusercontent.com/108508730/224329800-6404be27-8264-4ed6-860e-ef8e004fa67b.png)
    
    링크: [https://chrome.google.com/webstore/detail/webchatgpt-chatgpt-with-i/lpfemeioodjbpieminkklglpmhlngfcn/related?hl=ko](https://chrome.google.com/webstore/detail/webchatgpt-chatgpt-with-i/lpfemeioodjbpieminkklglpmhlngfcn/related?hl=ko)
    
    ChatGPT 사용 시 인터넷 검색 결과를 함께 전달해주는 확장 프로그램 입니다.
    
    ChatGPT는 2021년 까지의 자료만을 학습해서 최신 정보와 다른 답을 주는 경우가 있는데, 이 확장을 사용하면 ChatGPT로 질문 시 해당 키워드의 검색 결과를 인터넷에서 검색하여 함께 전달해 줍니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329845-5de70cad-4e5b-4279-a72d-4f8957d7e17e.png)
    
    예를 들어, MariaDB의 최신 버전은 몇인지 질문하면
    
    ![image](https://user-images.githubusercontent.com/108508730/224329894-a300b90f-6e5f-40ca-a99a-d1ce731c076c.png)
    
    인터넷에서 검색을 하여 나온 결과를 함께 전달해주고, 이를 바탕으로 ChatGPT가 아래와 같이 답변합니다.
    
    ![image](https://user-images.githubusercontent.com/108508730/224329933-584f702b-a6c3-49f2-ba9c-7bf2dc460564.png)
    

번외: ChatGPT for Google

![image](https://user-images.githubusercontent.com/108508730/224329986-a53f4229-b33b-4b11-a14d-09b2d45cd0ad.png)

[https://chrome.google.com/webstore/detail/chatgpt-for-google/jgjaeacdkonaoafenlfkkkmbaopkbilf/related?hl=ko](https://chrome.google.com/webstore/detail/chatgpt-for-google/jgjaeacdkonaoafenlfkkkmbaopkbilf/related?hl=ko)

Google에서 검색을 하면 ChatGPT로 검색한 결과도 함께 띄워주는 확장프로그램 입니다.

아래와 같이 구글에서 검색을하면  ChatGPT의 답변도 함께 띄워줍니다.

![image](https://user-images.githubusercontent.com/108508730/224330041-f6f0e388-f127-4587-a607-a0b0e8632156.png)

### 감사합니다!
