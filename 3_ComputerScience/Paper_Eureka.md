
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

- Reward design은 LLM에 제공하기 위해서는 환경세팅이 필요하다. 저자들은 context형태로 raw하게 LLM에게 RDP 의 M을 제공했다. 

- 저런 이유가 자유롭게 주어야 LLM에서 자기가 프로그래밍을 학습한 방식의 syntax 나 코드스타일을 건드리지 않는데 그래야 