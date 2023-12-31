
날짜 : 2023-10-25
태그 :   #AI #ComputerGraphics #HandAnimation 
출처 :   
저자 :   
url :   
인용 :   
위치 :  
연결문서 :   


# 개요

- 빠르게 Eureka 파트 구현 부분 분석하기 위해서 논문 읽고 정리 

# 내용

## 1. abstract and Introduction

- Eureka의 기여는 다음과 같다.
	1. reward function을 LLM 을 통해서 human-level performance로 늘려왔다.
	2. 수동적 reward function설계로 어러움을 겪는 dexterous manipulation task를 해결했다.
	3. 새로운 gradient-free in-context learning approch를 강화학습에 접목시키게 되었다.
	   
- 2023년의 L2R using LLM과 다르게 EUREKA는 task, reward template에 자유롭다 (few-shot examples)

- LLM 은 GPT-4 로 사용함 (source code를 보면 gpt-3.5 turbo도 되긴함)

- Eureka의 일반성은 3개의 key algorithm 의해서 가능하게 되었다.
	1. environment as context : environment source code를 context화 시켜서 zero-shot generate executable reward function을 LLM backbone으로부터 만들 수 있다.
	2. evolutionary search : EUREKA는 반복적으로 참여하는 reward의 배치를 실행하고 가장 괜찮은걸 정제함
	3. reward reflection : 이러한 in-context improvement는 reward reflection을 통해서 이루어진다 reward reflection은 reward function의 policy training statistic 에 기반한 문자로 된 요약이라고 보면 된다.

- EUREKA는 intermdediate rewards를 평가한다.  이걸 IsaacGym을 이용해서 해결한다. 

## 2. Problem Setting과 정의


- reward design의 목표는 실제 reward function이 직접 최적화하기 어려운 function을 design하는 것이다.
- 그래서 singh 의 논문에서 가져온 reward function을 논문에서는 소개한다.

### Reward Design Problem 

- tuple P =  < M, R, pi, F> 로 이루어져있다.
- M = ( S, A, T ) world model : S 는 state space , A 는 Action space , T 는 transition function
- R 은 reward function A() R -> bigPi 는 learning algorithm 최적화된 Reward function을 내놓는 policy pi를 주는 거라고 생각하면 된다. 
- F 는 policy에 대한  scalar evaluation 를 제공 해주는 fitness function이라고 한다.
	- 여기서 policy는 policy queries 를 통해서 제공한다는데... 이파트는 잘 모르겟다.

- 결국 RDP의 목표는 최적화된 Reward function을 제공하는 거고 그 판단은 R에 의해서 최적화된 policy를 Fitness 함수의 scala evaluation을 이용해서 한다고 보면 된다.

### Reward Generation Problem 

- 논문의 저자는 RDP의 problem setting을 code로 구현했다고 한다
- 어떤 일을 시키는 string l을 넣으면 RGP는 위에서 말한 RDP가 원하는 결과물 R을 제공해 준다.


## 3. Method

- introduction에서 말한거 3가지의 algorithm적 구성요소를 설명한다.
- 논문의 appendix A 에서 all prompt가 제공됨

### 3.1 Environment as context

- Reward design은 LLM에 제공하기 위해서는 환경세팅이 필요하다. 저자들은  raw environment code를 context로  LLM에게 RDP 의 M을 제공했다. 

- 저런 이유가 자유롭게 주어야 LLM에서 자기가 프로그래밍을 학습한 방식의 syntax 나 코드스타일을 건드리지 않는데 그래야 더 강력한 code generation을 준다고 한다..(솔직히 걍 말 지어낸거같은데)
- 그리고 근본적인 이유는 environment source code 는 형식적으로 환경이 무엇을 내포하고 무슨 변수가 사용되는 지만 알려준다고 함 
- 이걸 최대한 이끌어내기 위해서  EUREKA는 생 python 코드만 주고 보상함수나 포멧에 관해서는 단순하게 팁만 줌 (예를 들어 output으로 주어야 할 것들에 대해서)

- Appendix D를 통해서 관찰하면 될듯

- 이런 적은 instruction이 EUREKA의 zero-sho generate를 해결할 수 있는 시야로 보게 만든다

- FIg 3을 보면 EUREKA의 output을 보게됨

- EUREKA는 environment에서 제공하는 변수들을 활용하고 reward template나 환경에 특화되지않은 코드를 줌

- 하지만, 대부분의 reward는 구동조차 되지 않거나 Fitness function F에 최적화가 전혀 되지 않는다

### 3.2 Evolutionary Search

- 3.1 에서 생긴걸 다루기 위해서 evolutionary search가 나옴

- 각 iter에서 sample들은 서로 iid하다 그러기 때문에 sample을 늘리면 reward function을 낮춘다. 

- 실험해봤는데 충분한 sample 들을 줘봐도 끽해봐짜 1개의 executable 한 결과물을 준다. 

- Eureka는 전 iteration의 excutable한 결과물의 피드백을 이용해서 새로운걸 낸다. 

- 이걸 간단하게 특정한 mutation operator 로서 아카이브 하고 이거가 영향을 미친다고 한다.

- 이 mutation을 이용해서 EUREKA는 LLM로부터 결과물을 내놓는다. 이 최적화는 특정 iteration 수까지 반복된다 

- EUREKA 는 5번의 environment를 실행하고 16번의 샘플을 사용함.

### 3.3 Reward Reflection

- in-context reward mutation을 하기 위해서 
