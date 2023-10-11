
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
	- writer = SummaryWriter(...) (tensorboard 관련함수)
	- log_dir = log dir 설정
	- envs = make_vec_envs(...) 환경 세팅하는 함수 (gym_gang 의 envs.py 참고)
	- actor_critic = Policy(...) policy클래스로 init (gym_gang 의 model.py 참고)
	- if문 (line 91 to 113) RL 알고리즘에 따라 알고리즘 세팅 다르게함 (A2C, PPO, acktr)
	- if문 (line 115 to 129) gail에 대한 세팅 https://rlwithme.tistory.com/4 참고
	- rollouts = rolloutstorage 클래스 초기화 하는 느낌인데.. 이 부분은 따로 분석해야함
	- for문 (line 148 to 278) 까지는 이제 학습을 시작하는 라인이다.

- for문 (line 148 to 278)
	- if문 (149 새 153) lr decay 