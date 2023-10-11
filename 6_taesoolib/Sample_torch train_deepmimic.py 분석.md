
날짜 : 2023-10-11
태그 :  #taesoolib #RL #deepmimic
출처 :   
저자 :   
url :   
인용 :   
위치 :  
연결문서 : [[taesooLib_documentation_main]]


# 개요

- train deep mimic 코드 분석

# 진행

- sys.path.append(os.getcwd()) 아래의 import 되어있는 부분은 아마도, gym_gang사용하려고 그러는듯?

- luamodule.py를 읽어보면 sample python에서 lua를 사용하는 규칙에 대해서 알 수 있을듯?

- main()
	- get_args() 를 통해서 파싱된 프리셋 저장 arguments.py 보면 이해가능
	- torch.manual_seed 에 args.seed 줌 (난수 생성 일정하게)
	- 