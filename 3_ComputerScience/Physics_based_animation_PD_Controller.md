

날짜 : 2023-10-17
태그 :   #PBA
출처 :   
저자 :   
url :   https://web.cs.ucla.edu/~dt/papers/siggraph01/siggraph01.pdf
인용 :   
위치 :  
연결문서 :   


# 개요

- physics based animation 에서 pd servo가 뭘 의미하는지 고민해보장

- 애초에 pd controller는 pid controller에서 Integral이 빠진 부분을 의미하는 느낌이다.

- 그럼 PID Controller는 무엇인가? 

- PID Controller : https://ko.wikipedia.org/wiki/PID_%EC%A0%9C%EC%96%B4%EA%B8%B0

	- 들어오는 설정 값과 참조 값 사이의 오차를 비교해서 오차를 이용해서 제어를 할 수 있게 도와주는 제어기법
	- 오차값 + 적분값 + 미분값을 합쳐서 구성되어있다.
	- 폐쇠루프 시스템이다
	- 현재 컴퓨터 애니메이션에서는 I를 빼고 PD controller 두 개만 이용한다.

- physics based animation에서 PD controller를 사용하는 이유?
	- physics  reference motion을 따라하기 위해서.


https://arpspoof.github.io/project/spd/spd.html


