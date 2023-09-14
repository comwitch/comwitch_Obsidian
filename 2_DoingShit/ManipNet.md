
날짜 : 2023-09-13
태그 : #paper
출처 : 
저자 :
url : https://research.facebook.com/file/593672515280443/ManipNet-Neural-Manipulation-Synthesis-with-a-Hand-Object-Spatial-Representation.pdf
인용 : 
위치 : 
연결문서 :  [[Hand_Animation_Paper]]


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

-  메인 아이디어는 손과 물체 사이의 공간적 관계의 특성을 사용하는 것 (biomechanice literature 관점으로 가는 듯)

- coarse representation for the global object shape -> dense representation for local geometric details of the object surface whend  hand is in close proximity (근접함에 따라서 detail을 변화시킨다.)

- represent 3D geometry for neural networks 는 다음과 같은 방식들이 주어져 있다.
	- voxel occupancy grids
	- signed distance fields
	-  and point samples
	- 저자는 중요 정보들을 캡처하면서 feature dimention을 줄인다고 한다.( overfitting을 피하기 위해.)

- 저자는 저해상도의 voxel occupancy grid 사용해서 object shape를 표현한다.  여기에 손과 물체 표면 사이에의 distance samples가 디테일을 잘 캡쳐하는 low dimensional signal이다 

- finger pose 의 예측은 neural network를 통해서 배웠다.

- Control signals are 6D trajectories of the wrist and of the object

- deep learning 로 일반화 잘하기 위해 덜 애매한 input representation를 해야했고,  그 결과가 한개의 손 (오른손) - 물체 이 조합으로 건들게 했다. 그다음 다른 손과 물건을 학습시키는 것을 시킴

- 차주는거 성공했다함
![그림](../AttachedFiles/pic1.png)


요약하자면 다음과 같다

• A neural network-based motion synthesis system that can
generate detailed finger motions for one-/two-hand dexterous
object manipulation,
• a representation of hand-object spatial relation that enables a
neural network to generalize manipulation motions to a wide
range of object shapes and manipulation tasks,
• a hand-object interaction motion dataset that includes de-
tailed finger motions and dexterous manipulations of 16 man-
made objects


## System OverView

- 전체적인 시스템은 위에서 설명한 거를 조금 더 요약한 느낌이다.

### input
-  manipulated될 wrist와 object의 trajectories
- skined mesh (손, object)
- 오브젝트의 3D geometry
### output
- finger poses

- 아까 말했듯이 ManipNet은 한 손에 대해서 학습하고 결과물을 도출 할 수 있다. 다른 손은 어떻게 작동되어지는지는 section 4로.

### Sensing spatial relationships

- 네트워크의 일반화와 overfitting 줄임을 위해서 3개의 virtual sensor를 사용했다. 이 센서들은 물체의 모양을 단순한 복셀 그리드와 기하학적 point sample로 표현함.

- global features가 전체적인 포즈와 미래 동작을 예측한다면, local feature는 기하학적 다형성을 위한 일반화를 가능하게 한다.

### manipnet (deep neural network)
- autoregressive model

- 시계열 +RDN(residual dense
network )

- 인풋은 위에서 말한 previous frame 포함 pose, sensor feature, control signals(wrist 와 object의 trajectories)

- 예측하는 것 : 손가락과 물체 간의 거리 + 새로운 손 포즈 

## Right hand-centric coordinate system

- 이 사람들은 input trajectory의 바로 전 프레임의 오른손 손목에 coordinate system을 적용했다.

- 모델로 들어가기 전, 모든 데이터들을 저 coordinate system으로 변환부터 한다.

- 왼손은 단순하게 데이터 들어오면 반대로 mirror 시켜버린다. (그러면 오른손이 한 것처럼 보이기 때문) 그리고 결과물 나오면 다시 오른손으로 바꾸어 버린다.

- interaction하는 물건이 하나만 있다면, 오른손-물건 한번 manipnet 보낸 다음 왼손-물건 manipnet으로 보냄

- 결국 저자는 오른손만 학습시키면 된다.

- 왼손잡이 오른손잡이에 의해서 생길 수 있는 편향성은 input trajectories에 나올 수 있다고 함.

- 한 손으로만 테스트 하는 것이 결국 coordinate문제까지 고려하면 가장 최적화된 결과다 라고 저자는 생각함 

- object local coordinate 기준으로도 생각해보았는데, 여러 물건 사이로 가면 문제가 생긴다.


## Sensing the spatial relations between hands and objects

-  저자는 마르고 닳도록 말했지만, 3가지의 sensor가 존재한다.

- 이 센서들로 얻어진 local 한 정보들 때문에 글로벌한 shape을 얻을 수 있다.

### Ambient Sensor

-  18 * 18 * 18 cm3 의 voxel grid가 존재 (해상도는 10 10 10)

- 손목에 붙어있음 meddle finger의 origin에 뭍어있다. grid의 센터는 constant offset( x = 4 y= 0 z = 9)를 가지고 있다.

- voxel cell의 occupancy percentage는 다음과 같이 구한다.

	-  0 (no instersection)
	- 1 - d/s (intersection) (d : d(centercell, object), s : 1.8)

- voxel cell의 해상도를 늘리면 더 자세한 물체의 표면을 알 수 있지만, 계산복잡도가 확 올라버리기 때문에, distance-based sensor들을 사용한다. 

### Proximity Sensor

- 104개의 손과 손바닥의 sensor가 가까운 물건들을 탐지하게 유니폼리하게 배치됨 normal direction 방향으로 ray를 내놓고 분석한다 (threshold = 10cm)

- sensor가 이미 object안에 있다면, 방향을 바꿔서 바꾼다.

- distance feature는 다음과 같이 계산되어진다.
	- dj = sign(pj)||pj - pc||  (ray-object intersection)
	- dj = sign(pj) 10(threshold) (no instersection)

- pj는 센서 포지션, pc는 ray와 object 표면에 닿은 점

- feature vector D 는 이러한 dj들을 모아놓은 것이다.

- proximity sensor는 물체와 손이 가까워 질 때 물체의 세부 표면을 알 수 있다.


### Signed Distance Sensor

- Signed Distance seonsers는 object의 signed distance를 모은다. (각각 22개의 finger joint로부터 찾는다 )
- 가까운 distance value랑 normal direction 두 개를 가지고 있다.

- hand joint j에서의 특징값 sj는 다음과 같이 계산된다.
	- sj = {sign(pj) min(||pj - po||, threshold), no}
	- pj = joint position, po = closet object position, no = normal vector

- feature vector S 는 이러한 sj들을 모아서 사용한다. 

- signed distance field of an object 는 미리 컴파일되어 있다. 

- 저자는 surface normal vector 