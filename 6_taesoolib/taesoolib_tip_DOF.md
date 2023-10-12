
날짜 : 2023-10-12
태그 :   #taesoolib 
출처 :   
저자 :   
url :   
인용 :   
위치 :  
연결문서 :   [[taesooLib_documentation_main]]


# 잡설

pose를 결정할 때 우리는 SetDOF 에 vectorn 을 넣어서 설정을 하는데 그 벡터의 크기를 궁금해 할 수 있다.

다음을 생각하면 된다.

벡터의 크기 = 조절할 수 있는 관절 수 + 이동할 수 있는 벡터 수

여기서 회전 할 수 있는 관절 수 는 wrl의 JointAxis 의 개수의 총 합을 보면 되고, 이동 할 수 있는 본의 수는 'jointType' 이 "free" 인 본의 개수를 보면 된다.

