
날짜 : 2023-10-25
태그 :   #AI #ComputerGraphics #HandAnimation 
출처 :   
저자 :   
url :   
인용 :   
위치 :  
연결문서 :   


# 개요

- 빠르게 Eureka 파트 구현부분 분석하기 위해서 논문 읽고 정리 


# 내용

## abstract and Introduction

- Eureka의 기여는 다음과 같다.
	1. reward function을 LLM 을 통해서 human-level performance로 늘려왔다.
	2. 수동적 reward function설계로 어러움을 겪는 dexterous manipulation task를 해결했다.
	3. 새로운 gradient-free in-context learning approch를 강화학습에 접목시키게 되었다.
	   
- 2023년의 L2R using LLM과 다르게 EUREKA는 task, reward template에 자유롭다 (few-shot examples)

- LLM 은 GPT-4 로 사용함 (source code를 보면 gpt-3.5 turbo도 되긴함)

- Eureka의 일반성은 다음에 의해서 가능하게 되었다.
	-1