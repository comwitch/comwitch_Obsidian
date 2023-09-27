
날짜 : 2023-09-27
태그 : #taesoolib 
출처 :  
저자 :   
url :   
인용 :   
위치 :  
연결문서 :   


# 개요
- 물리 시뮬레이션을 위한 충돌 테스트



# 내용
- 중요한거 기준으로 test

### model_files.hyunwoo=deepCopyTable(model_files.default)
- do ... end 구문은 모델을 등록하는부분

### EVR:onFrameChanded(win, iframe)
- local에서 model.rendering_step/model.timestep+0.5 이놈의 floor를 계산한 이유?
	-> maybe 반올림
- 