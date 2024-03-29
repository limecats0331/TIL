## 1-2 컴퓨터 구조의 큰 그림

컴퓨터의 구조는 크게 두가지로 나눌 수 있다.
* 컴퓨터가 이해하는 정보
	* 명렁어 - 데이터를 움직이고 컴퓨터를 작동시키는 정보
	* 데이터 - 컴퓨터가 이해하는 정적인 정보
### 컴퓨터의 네 가지 핵심 부품

* 중앙처리장치(`CPU`)
	* 메모리에 저장된 명령어를 읽어서 해석하고 실행하는 부품
	* 내부 구성 요소 중 중요한 3가지
		* 산술논리연산장치(`ALU`) - 계산만을 위한 부품, 대부분의 계산을 도맡아 수행한다.
		* 레지스터 - `CPU` 내부의 임시 저장 장치. 여러 개의 레지스터가 존재하고 각기 다른 이름과 역활을 가지고 있다.
		* 제어장치(`CU`) - 제어 신호(`control signal`)라는 신호를 내보내고 명령어를 해석하는 장치
			* CPU가 메모리에 값을 읽고 싶을 때는, `메모리 읽기`라는 제어 신호를 보낸다.
			* CPU가 메모리에 값을 쓰고 싶을 때는, `메모리 쓰기`라는 제어 신호를 보낸다.
* 주기억장치(`memory`)
	* 현재 실행되는 `프로그램의 명령어와 데이터를 저장`하는 부품. 프로그램이 실행되려면 `반드시 메모리에 저장`되어야 한다.
	* 메모리에 저장된 값에 빠르고 효율적으로 접근하기 위해서 `주소`라는 개념이 사용된다.
* 보조기억장치
	* 주기억장치는 가격이 비싸서 저장 용량이 적고, 전원이 꺼지면 저장된  내용을 잃는다는 단점이 있다.
	* 메모리보다 크기가 크고, 전원이 꺼져도 내용을 잃지 않는 메모리를 보조할 장치
	 * `HDD`, `SSD`, `USB Driver`, `DVD`, `CD-ROM` ... etc
* 입출력장치
	* 보조기억장치는 관점에 따라 입출력장치의 일종으로 볼 수 있다.

#### 대략적인 흐름
1. 제어장치는 메모리의 1번에 저장된 명령어를 읽어 들이기 위해서 `메모리 읽기`라는 신호를 보낸다.
2. 메모리는 해당 신호를 받아 CPU에 1번에 저장된 명령어를 건내준다. 
3. 해당 명령어는 `레지스터에 저장`되고 제어장치가 명령어를 해석한 뒤에 2번과 3번에 저장된 데이터가 필요하다고 판단한다. 
4. 제어장치는 2번과 3번에 저장된 데이터를 읽어오기 위해서 `메모리 읽기` 라는 신호를 보낸다.
5. 메모리는 2번과 3번에 저장된 데이터를 `CPU`에 건내주고, 이 데이터들은 서로 다른 레지스터에 저장된다.
6. `ALU`는 읽어온 데이터로 연산을 수행하고, 결과를 또 다른 레지스터에 저장한다.
7. 연산이 끝나면 연산이 끝났음으로 다음 명령어를 수행하기 위해서 `메모리 읽기`라는 제어 신호를 보낸다.
8. 메모리는 명령어를 `CPU`에 저장하고 이 명령어는 명령어를 저장하는 레지스터에 저장한다.
9. 제어장치가 명령어를 해석한 뒤에 결과를 저장해야된다고 판단한다.
10. 저장하기 위해서 메모리에 `메모리 쓰기`  신호와 함께 레지스터에 저장되어 있던 결과값을 보낸다.

#### 메인보드와 시스템 버스
지금까지 설명한 컴퓨터의 핵심 부품은 모두 `메인보드`라는 판에 연결된다. 메인보드를`마더보드`라고도 부른다. 메인보드에는 여러 부품을 연결할 수 있는 슬롯과 연결 단자가 있다.  

메인보드에 연결된 부품들은 서로 정보를 주고 받을 수 있는데 이는 내부의 `버스`라는 통로가 있기 때문이다. 컴퓨터 내부에는 다양한 버스가 있지만 그중에 가장 중요한 버스는 `시스템 버스`이다.

시스템 버스는 주소 버스, 데이터 버스, 제어 버스로 구성되어 있다.
* `주소 버스` : 주소를 주고 받는 통로
* `데이터 버스` : 명령어와 데이터를 주고 받는 통로
* `제어 버스` : 제어 신호를 주고 받는 통로

위의 흐름에서 보면 `CPU`가 메모리 속 명령어를 읽기 위해서 제어 장치에 `메모리 읽기`라는 신호를 보낸다고 했는데, 이때 `CPU`는 제어 신호만 보내지 않고 `제어 버스`로는 `메모리 읽기`라는 신호를 보내고, `주소 버스`로 읽고자 하는 주소를 보낸다.

어떠한 값을 저장할 떄는 `데이터 버스`를 통해서 `저장할 값`을, `주소 버스`를 통해서 `저장할 주소`를, `제어 버스`를 통해서 `메모리 쓰기`라는  신호를 보낸다.