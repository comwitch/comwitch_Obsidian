
태그: #paper
출처: 
저자:
url:https://research.facebook.com/file/593672515280443/ManipNet-Neural-Manipulation-Synthesis-with-a-Hand-Object-Spatial-Representation.pdf
인용:
위치:
연결문서: [[Hand_Animation_Paper]]


# 개요

- ManipNet이 무엇인지에 대해서 좀 리뷰를 해야 함.

- ManipNet의 code 테스트를 해야함


# 내용

## Introduction

-  we choose to learn natural mainulation behaviors directly from data using a deep neural network given the explosive success of deep learning
	- 딥러닝을 이용해서 자연스러운 행동 배합을 배우겠다는 뜻.

- manipulation은 단순한 물건의 특징(모양, 크기, 기능) 뿐만 아니라 의도 , hand anatomy 심지어 사람의 선호도에 따라서 다름 

- Real-time Hand-object interaction motion capture는 가능하지만, 넓은 공간에서의 캡쳐는 여전히 불가능함

- 저자는 적은 수의 물건 모양을 학습 시키는 것을 통해 기하학적  다형성을  배우는데 초점을 맞추고 그것을 목적이 있는 물체를 잡는 애니메이션을 넘어 사람처럼 자연스럽고 의도하지 않는 방법으로 하는 것을 목표로 확장시키는 것에 초점을 맞춘다.

-  메인 아이디어는 