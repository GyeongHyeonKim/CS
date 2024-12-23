### 배경지식

---

![](https://velog.velcdn.com/images/sujipark2009/post/d6e125ed-0467-45c5-8228-6070f604a46b/image.png)

위 그림은 한 프로세스의 일생을 시작부터 종료까지 나타낸 것이다.

프로세스의 일생에는 CPU에서 기계어를 실행하는 단계고 있고, I/O를 하는 단계가 있다.

기계어를 실행하는 단계를 `CPU Burst` , I/O를 하는 단계를 `I/O Burst` 라고 한다.

명령어를 Load,Store하는 이런 작업은 다 CPU에서 수행하는 작업이다.

또 어떤 프로세스는 CPU를 오래쓰다가 I/O를 하기도 하고, 또 양쪽의 작업을 조금씩 하는 프로세스도 존재한다.

![](https://velog.velcdn.com/images/sujipark2009/post/ac265d2e-72f9-480d-b208-4475b1975b03/image.png)

프로그램을 2가지로 나누어서

`CPU Bound Job`,`I/O Bound Job` 이라고 부른다.

둘 다 CPU를 쓰고싶어하는데, 누구에게 CPU를 먼저 줘야할까?

먼저 도착한 프로세스에게 주어야하는가?

이 방법에는 여러가지가 있고, 그 방법들이 바로 **CPU 스케쥴링**이다.

시스템안에 이런 이질적인 작업들(I/O를 쓰는 작업,CPU를 쓰는 작업)이 섞여있지 않다면 CPU 스케쥴링이 그렇게 주요한 문제는 아닐것이다.

그런데 CPU를 오래 쓰려는 프로세스하고, CPU를 짧게쓰고 I/O를 하려는 프로세스가 섞여있다면 누구에게 빨리 CPU를 주는게 좋을까?

I/O Bound인 프로세스에게 주면 잠깐쓰고 I/O를 하러 가겠지?

또 I/O는 주로 사람과 Interaction을 하는 작업이기 때문에, 여기에 CPU를 빨리 얻을 수 있게 해주는 방법이 필요하다.

반면 CPU를 오래쓰는 작업은 복잡한 작업을 하는데 하루 이상이 걸릴수도 있다.
그럼 5초있다가 CPU를 준다고 해도 크게 달라지는건 없을것이다.

CPU를 길게쓰는 프로세스에게 CPU가 넘어가서, I/O Job이 일을 못하면, I/O 장치도 놀게되서 전체적인 효율성이 떨어지게 된다.

![](https://velog.velcdn.com/images/sujipark2009/post/64f3b7fe-5e24-4e61-a3c6-033577034228/image.png)

**I/O Bound Process**

- Many Short CPU burst

CPU burst란, CPU를 연속적으로 쓰는 시간인데, 그게 짧다는 뜻이다.

**CPU Bound Process**

- Few very long CPU Burst

드물지만 연속적인 CPU작업을 매우 길게 수행한다는 것

![](https://velog.velcdn.com/images/sujipark2009/post/ab8bafba-d541-49e3-9491-f7f643150536/image.png)

어쨌든,, 그래서 CPU 스케쥴링이 필요하다는건 알겠다.

2가지 용어가 있는데 **Scheduler**와 **Dispatcher**가 있다.

이건 HW일까 SW일까 프로세스일까...

전에도 스케쥴러가 나왔는데 이건 소프트웨어이고 어떤 프로세스를 의미하는게 아니라 OS안에 있는 스케쥴링을 위한 코드를 그냥 스케쥴링이라는 역할을 하기 때문에 네이밍을 해놓은 것이라고 했다.

`CPU Scheduler` 는 OS코드중에서 스케쥴링을 하는 코드를 말한다.

- 누구에게 CPU를 줄 지 결정

`Dispatcher` 는 실제로 그 결정된 프로세스에게 CPU를 넘기는 역할을 한다.

CPU가 A라는 프로세스로부터 B라는 프로세스에게 넘어가는걸 `Context Switching` 이라고 했는데,

이런 문맥 교환이 되려면 CPU를 빼앗기는 프로세스의 Context를 저장하고 새롭게 CPU를 얻는 프로세스의 Context를 Load하는 작업이 필요한데, **이런 작업을 Dispatcher가 하게 되는 것이다**

CPU스케쥴링이 일어나면, CPU의 상태변화가 어떻게 되느냐면..

일단 CPU를 얻으면 Ready -> Running 상태로 넘어가게된다.

- Ready Queue에 프로세스들이 줄을 서 있다가 CPU를 얻으면 Running이 되는 것

그럼 CPU를 놓는 과정은 어떻게 될까?

- 계속 쓰고 싶은데 빼앗겨서 Ready Queue로 들어가는 경우
- I/O를 해야해서 Blocked 상태로 들어가는 경우

이렇게 있을 것이다.

1. Running -> Blocked의 경우는 CPU를 쓰다가 I/O 같은 오래 걸리는 작업을 요청하는 경우

2. Running -> Ready는 계속 CPU를 쓰고싶지만, 그러면 다른 프로세스의 응답시간이 느려지니까 `Timer interrupt` 가 걸린 것

3. Blocked -> Ready는 I/O 완료 후 인터럽트가 걸린 것

- 만약 I/O를 하러갔던 프로세스가 대단히 우선순위가 높다면, I/O가 끝나자마자 바로 CPU를 얻을 수도 있다.

4. Terminated는 프로세스가 종료되었을 때

여기서 1번과 4번을 `Nonpreemptive`라고 하고 나머지는 `Preemptive` 라고 부른다.

**Preemptive**는 강제로 빼앗는다는 의미이고, 자진해서 내놓기까지는 빼앗기지 않는다가 **Nonpreemptive** 이다.

이게 무슨말이냐..

1번과 4번은 I/O를 하게되면 더이상 CPU가 필요없으니 자진해서 CPU를 내놓는 것이다.

또 프로세스가 끝나면 필요없으니 자진해서 내놓는 것.

나머지는 CPU를 계속 쓰고 싶은데, 정책상 다른 프로세스들도 써야하기에 빼앗기는 것이다.

이렇게 CPU 스케쥴링 방법은 **비선점이냐 선점이냐의** 2가지로 나뉜다.

![](https://velog.velcdn.com/images/sujipark2009/post/95b2423e-ae12-452e-9320-f12a2316359c/image.png)

이렇게 있는데.. 이건 조금 있다가 정리하도록 하고

우선 CPU 스케쥴링의 **성능 척도**에 대해서 알아보자.

![](https://velog.velcdn.com/images/sujipark2009/post/05f262df-4fc7-4815-a895-8b8c5a3530ce/image.png)

#### Scheduling Criteria

---

저런 스케쥴링 알고리즘 중에 어떤 것이 좋은지 판단할 수 있는 방법이 있어야 하겠지?
그런 방법을 성능척도라고 부른다.

크게 3가지로 구성되어있다

#### CPU Utilization(이용률)

---

전체 시간 중에서 CPU가 놀지않고 일한 시간의 비율을 말한다.

100%인게 좋을까 50%인게 좋을까?

CPU는 대단히 비싼 자원이므로, 놀리지 않고 일을 시키는게 훨씬 좋은것이다.

중국집을 차려서 주방장(CPU)에게 요리를 시키는데, 주방장이 쉬지않고 일을 많이하면 좋은것처럼..

똑같이 손님이 왔는데, 요리를 하는 순서에 따라 주방장이 일하는 시간이 달라질 수 있다.
그게 바로 스케쥴링에 따라 CPU 이용률이 달라질 수 있는 것이다.

#### Throughput(처리량)

---

단위 시간 당 처리량이다.

시스템 입장에서 보는 척도로, CPU가 얼마나 많은 일을 했느냐이다.

주방장이 하루동안 몇 명의 손님을 받았는가..라고 할 수 있다.
(단위시간 당 프로세스를 몇 개 완료시켰는가)

#### Turnaround time(소요시간,반환시간)

---

사용자에서 시간이 빠를수록 좋은 것.

내가 밥먹고 기다리는 모든 시간을 합친 것이다.

CPU 대기시간 + 실제로 CPU를 쓰는 시간을 합친 것

#### Waiting time(대기시간)

---

CPU를 쓰러와서 기다린 전체 시간.

기다린 시간의 총합이다.

CPU를 얻었다가 뺏기고 또 줄을 서면 기다리는 시간이 생긴다.

Waiting time과 Response time의 차이는, waiting time의 경우 프로세스가 CPU를 얻었다가 다시 뺏기고 또 기다릴 수 있는데 이 시간까지 더한 것이다.

그러나 Response time은 처음으로 CPU를 얻을 때까지의 시간만 의미한다.

#### Response time(응답시간)

---

어떤 프로세스가 CPU를 쓰러 들어와서 최초로 CPU를 얻기까지 걸리는 시간이다.

혼동하면 안되는건,

CPU를 쓰러 들어와서 최초로 CPU를 얻기까지 걸리는 시간으로, 처음 얻고 바로 뺏길수도 있다. 그 다음에 다시 CPU를 얻는 시간은 관심사가 아니며 오로지 `최초`에만 관심이 있다.

#### 중국집에 비유한 Criteria

---

CPU Utilization은 전체 운영시간 중 주방장이 일한 시간이다.

Throughput은 운영시간 중 주방장이 몇 명의 고객에게 음식을 만들어 냈는가로 비유할 수 있다.

Turnaround time은 고객이 중국집에 들어와서 식사를 마치고 나갈때까지 걸린 시간의 총합이다.

Waiting time은 코스요리를 시킨다고 했을 때, 대기 시간의 총합이다.
코스요리는 한번에 모든 요리가 나오는 것이 아니라, 중간중간에 요리가 나오기 때문에 중간 대기시간이 생긴다.

Response time은 첫번째 음식이 나올때까지 기다린 시간이다.

#### FCFS(First-come First-served)

---

말 그대로 선착순이다.

![](https://velog.velcdn.com/images/sujipark2009/post/ae20a7fa-2444-484b-a2b3-aa184d8fa4f5/image.png)

**비선점**방식으로, 먼저 온 프로세스에게 CPU를 주고 끝날때까지 기다리는 것.

인간세상에서는 공정하지만, CPU에게는 공정하지 않다.

프로세스가 도착한 순서가 P1,P2,P3라고 했을 때, P1이 CPU를 다 쓸 때까지 뺏지않고 다 끝나야 P2,P3가 순차적으로 사용할 수 있다.

Waiting time을 보면, P1은 전혀 기다리지 않았다.
P2는 0초에 도착해서 24초를 기다렸고 P3는 0초에 도착해서 27초를 기다렸다.

Waiting time의 평균은 (0+24+27)/3 => 17초가 된다.

근데, 만약 CPU를 짧게쓰는 프로세스가 먼저 도착했었다면?

![](https://velog.velcdn.com/images/sujipark2009/post/2cecb1b5-8a33-4a09-890d-56d6547c64ea/image.png)

이 경우, Waiting time의 합은(0+3+6)/3 => 3초가 된다.

이전보다 훨씬 줄어들었다.

앞에 긴 프로세스가 들어와버리면, 전체가 길어지게 되는 것이다.

한 줄로 줄을 서서 기다릴텐데, 긴 친구 하나가 CPU를 가지고 있게 되면 나머지는 전부 기다리게 될 것이다.

이런 현상을 `Convoy effect` 라고 부른다.

Short porcess behind long process => 긴 프로세스가 먼저 도착해서 CPU를 쓰는 바람에 짧은 프로세스들이 오래 기다려야 하는 것.

#### SJF(Shortest-Job-First)

---

CPU를 가장 짧게쓰려는 것한테 먼저 주는 것.

대기중인 프로세스 중에 CPU를 가장 짧게 쓰려는 프로세스에게 먼저 주는 것이다.

기다리는 시간 측면에서 보자면, 가장 Optimal한 방법이다.

Optimal이라는건, 최적. 즉, 가장 좋은 방법이라는 의미이다.

다른 어떤 스케쥴링을 사용하더라도 SJF보다 대기시간의 평균을 더 짧게할수는 없다.

**비선점 버전과 선점 버전** 2가지로 나눌 수 있다.

비선점의 경우,

- 현재 큐에 줄서있는 것들 중에서 가장 CPU burst가 짧은 것을 꺼내서 돌리고 있는데, 그것보다 더 짧은 시간을 가진 프로세스가 들어오는 경우

그렇게 되더라도 이미 CPU를 줬으면 그 시점에 가장 짧다고 판단되어서 준 것이므로, 해당 프로세스가 다 쓰고 나갈때까지는 뺏지 않는 것

선점의 경우,

가장 짧은시간이라 줬는데, 큐에 마침 더 짧은게 들어온다면 더 짧은 친구에게 CPU를 주는 방식이다.

어떤게 더 Optimal한 방법일까? 어떤게 평균 대기시간을 더 짧게하는 방법일까?

빼앗는 방법이 더 Optimal하다.

선점방식의 SJF를 `SRTF(Shortest-Remaining Time First)` 라고도 부른다.

SRTF란, 남은 시간이 가장 짧은 친구에게 CPU를 먼저 준다는 의미이다.

5초동안 CPU를 쓰기로 해서 일단 줬다고 해보자.

3초를 사용하고 2초가 남은 시점에 1초짜리 프로세스가 하나 들어온다면, 남은 시간이 더 짧은 1초 프로세스에게 CPU를 줘야한다.

![](https://velog.velcdn.com/images/sujipark2009/post/86762a37-4560-45cc-a0b2-6be53cfbf6dd/image.png)

비선점 방식의 SJF이다.

프로세스가 4개가 있는데, 0초에는 P1만 있으니 P1이 다 돌고 그 다음부터는 P3,P2,P4 이런식으로 Burst time이 가장 짧은 순서로 실행된다.

![](https://velog.velcdn.com/images/sujipark2009/post/8eb6b16f-a719-40dd-a4d5-24435c8cd6c7/image.png)

이건 선점의 경우인데, P1이 돌다가 남은 시간이 더 짧은 P2가 들어와서 뺏기고, P2가 돌게되는 이런식으로 선점이 된다.

~~캬~ 이렇게 최적의 방법이 있으니 다른 스케쥴링은 안배워도 되겠다!!~~

하지만 SJF의 치명적인 약점이 2가지 있다.

1. 이 방법은 `Starvation` 을 발생시킬 수 있다.

너무 효율성만 생각하다 보니, 짧은 프로세스에게 무조건 우선권을 주도록 되어있다.

그래서 Long job은 영원히 CPU를 얻지 못하게 될 수도 있다.

2. 누가 짧게 CPU를쓰고.. 길게 쓰는지를 알 수 있나?

다음번의 CPU Burst time을 예측할수가 없다.

프로그램이 단순하지 않으니..if문도 섞여있고.. 여러가지 상황에 따라 다르기 때문이다.

그렇다면 SJF는 아예 쓸 수 없는 방법일까?

그렇지는 않다. 모르면 **예측**을 하면 된다.

어떻게 하느냐.

I/O Bound job과 CPU Bound job은 그 성격이 다르다.

그래서 이번에 CPU를 얼마나 쓸 지 정확히는 모르지만, 과거의 CPU Burst를 보고 이번에 얼마나 쓸 지를 예측할 수 있다는 것.

![](https://velog.velcdn.com/images/sujipark2009/post/370cbf7b-bfeb-4b25-b773-49986535f75f/image.png)

N+1번째를 예측하고 싶으면, N번째까지는 실제 기록이 있는거니까 그 N개의 데이터를 바탕으로 예측하면 된다.

N+1번째 예측값 = 직전의 CPU Burst길이 a + (1-a) \* 직전의 예측값

수식을 간단히 해보면.. 직전은 더 많이 반영하고 오래될수록 그 영향이 줄어들면서 다음 CPU Burst를 예측하게 된다.

어쨌든 SJF는 CPU Burst가 짧은 친구에게 먼저 주는건데, **Starvation**이 생길 수 있고 실제 CPU Burst를 알 수 없으므로 과거를 통해 **예측**을 해서 사용해야하는 문제점이 있다.

#### Priority Scheduling

---

![](https://velog.velcdn.com/images/sujipark2009/post/0dc7e7b6-aa1b-45e9-922a-5b108d427834/image.png)

우선순위가 높은 친구에게 먼저 CPU를 주겠다..는 것이다.

보통 우선순위 값을 정수로 표현하는데, 작을수록 더 높은 우선순위를 가진 것으로 판단한다.

이것도 **선점**,**비선점**으로 구현이 가능하다.

우선순위가 높다고 해서 CPU를 줬더니, 더 높은 우선순위의 프로세스가 왔을 때 뺏을지 안뺏을지..

SJF도 일종의 Priority Scheduling이다. 우선순위를 CPU burst time으로 본다면, 우선순위 방식으로도 볼 수 있다.

방금전에 봤던 SJF를 포함한 Priority Scheduling의 문제점 또한 `starvation` 이다.

우선순위가 낮으면 영원히 실행이 안될수도 있다.

이것에 대한 해결책은 `aging`인데, 말 그대로 오래 기다리면(나이를 먹으면) 우대를 해주자는 것.

기다린 시간만큼 우선순위를 높이고 높여서 언젠가는 우선순위가 높아져서 CPU를 쓸 수 있지 않겠는가.. 이것이 aging이라는 메커니즘이다.

#### Round Robin(RR)

---

![](https://velog.velcdn.com/images/sujipark2009/post/1da3d5b6-272a-48ca-ac93-e83b06574b37/image.png)

실제로 CPU스케쥴링에서 가장 많이 사용하는 방법의 근간이 되는 방식이다.

예전에 Timer가 붙어있으면서 CPU를 사용자 프로그램에게 넘길 때 Timer를 세팅하고 넘긴다고 했었는데, 이를 통해 CPU의 독점을 막을 수 있다고 했었다.

이 철학에 가장 잘 맞는게 Round Robin 방식이다.

프로세스가 CPU를 얻을 때는 `time quantum`을 가진다.

할당 시간이 지나면 프로세스는 CPU를 빼앗기고 큐에 들어간다.

I/O Bound job은, CPU사용시간이 짧으니까 본인의 할당 시간 내에 원하는 만큼 CPU를 쓰고 I/O를 하러 나가게 되고,

CPU Bound job은 CPU를 오래 쓰려는 프로세스이기 때문에 한번에 원하는 만큼 못 써서 CPU를 뺏기고..얻고.. 이 과정을 반복하게 된다.

무조건 I/O job에게 양보하는게 아니라 주어진 시간만큼 쓰고 나가게된다.

할당시간이 길면 FCFS와 동일하고, 너무 짧으면 Context switching으로 인한 오버헤드가 커진다.

그럼 이 time quantum은 어느정도로 해야할까?
I/O는 한번에 나가지만 CPU는 여러번 걸쳐서 해야하는 정도로 하면 적당..하다고 함

![](https://velog.velcdn.com/images/sujipark2009/post/c51c0a33-6c69-44d9-8836-6b58473b9392/image.png)

아까 SJF가 Waiting time을 가장 짧게한다고 했었는데, RR방식은 뭐가 좋은걸까?

바로, **Response time**이 더 짧다.

성능척도중에 시간과 관련된 3가지가 있었는데, 그 중 Response time이라는건 본인이 CPU를 쓰고자 하는 큐에 들어와서 `첫번째` CPU를 얻기까지 걸리는 시간이라고 했다.

이 시간이 빨라지면 뭐가 좋을까?

Time sharing 환경에서 Interactive 한 job들이 많은 환경에서 빠른 응답시간은 중요한 의미를 갖는다.

Round Robin을 사용하는 이유는, **Long job과 Short job이 섞여있기 때문**이다.

다 같은거만 있으면 RR을 쓰는게 어떤 효과를 발휘할까?

전부다 100초짜리가 있는데, 그걸 1초 할당시간씩 RR을 하게 된다면?

은행에서 10분씩 볼일을 봐야하는 사람이 100명이 왔는데, 그걸 FCFS로 처리하면 10분하고 나가고...10분하고 나가고.. 이렇게 되는데

10초씩 끊어가지고 이 사람 조금.. 저 사람 조금.. 이런식으로 하게 되면

볼일을 다 보고 나가는 시간이 엄청 늦어진다. 맨 마지막에 다같이 집에가게 되는 상황이 벌어진다.

한명씩하면 중간중간에 한명씩 집에 갈건데..

이게 Response time의 관점에서는 좋지만, Average turnaround time은 길어진다.

...

RR까지 살펴봤는데, RR은 사실 CPU 스케쥴링에 있어 굉장히 획기적인 방법이다.

CPU Bound job과 I/O Bound job의 성격을 미리 몰라도 `time quantum`을 통해 적절히 그에맞게 대우를 해주는게 그 특징이다.

RR스케쥴링은 Long job이 손해를 볼까 Short job이 손해를 볼까?

SJF의 경우, Long job은 그냥.. 죽으라는건데.. 극단적인 효율성을 추구하는 방식이다.

FCFS는 좀 운에따라서 먼저 도착한게 먼저 혜택을 받는 것이다.

RR은 Long job이나 Short job이나 합리적이다.

기다리는 시간이 본인이 CPU를 쓰는 시간에 비례하는 특징이 있다.

Short job이라고 하면, 대단히 짧은 친구는 CPU를 얻어서 한번에 할당시간안에 다 쓰고 I/O를 하러 나갈 것이다.

Long job이면 썼다가 뺏겼다가.. 이런 과정을 반복할것이다.

그걸 계산을 해보면, CPU를 5번에 걸쳐서 써야하는 친구는 큐에 5번 줄서서 기다린거고 한번의 분량만 써도되는 Short job은 한번만 기다려서 쓰고 나가게 되는 것이다.

지금까지의 알고리즘은 한 줄로 줄서는 거였는데, 지금 말할 `Multilevel Queue`는 여러줄로 서는 것이다.

#### Multilevel Queue

---

![](https://velog.velcdn.com/images/sujipark2009/post/a0698a08-6286-4a5c-b065-c092da15731d/image.png)

CPU는 한개인데, 줄은 여러개..

Ready Queue를 여러개로 분할한다.

- Foreground에는 Interactive한 Job들이 위치
- Background에는 Batch Job들(오래 CPU를 쓰는 그런것들)

각 큐는 독립적인 스케쥴링 알고리즘을 가진다.

극단적으로는 Foreground job이 있는경우, Background job을 실행시키지 않을수도 있다.

Foreground가 비었을 때, Background를 쓰게하는 `Fixed Priority Scheduling` 방식이 있다.

이건 좀 차별이 있는 방식이다.

Background 큐에 대해서는 starvation의 가능성이 존재한다.

그렇게까지 안하는 방법이 바로 `time slice` 방식이다.

무조건 Foreground에 우선순위를 주는게 아니라, 각 큐마다 어느정도 시간의 가중치를 주는 것.

Foreground에는 전체시간의 80%, Background에는 20%를 분배하면 Background가 starvation을 당하지는 않을 것.

그러면서도 Interactive 한 job들이 더 빨리 CPU를 얻을 수 있게 된다.

Foreground는 Interactive job들이 많으니, 응답속도가 빨라야한다. 그래서 **Round Robin** 방식을 사용하고

Background는 Long job이 위치하는 거니까.. 그것들은 CPU를 줬다뺏는게 더 비효율적이니 그냥 **FCFS**를 쓰면 되겠다.

![](https://velog.velcdn.com/images/sujipark2009/post/293549af-b92c-4f8e-bd64-51ce04fad96e/image.png)

이렇게 큐가 5개가 있는 그런 `Multilevel Queue` 가 있을수도 있다.

가장 우선순위가 높은 큐는, 시스템 프로세스들이 줄서는 곳이다.

그 다음이 사람과 상호작용을 하는 짧은것들.. batch...student..이런순이다.

큐는 이 중에 어느 한 곳에 줄을 서게되고, 그리고 그 신분으로 평생 살아가는 것이다.

약간 조선사회처럼... 신분의 변화가 없이 한번 큐에 들어가면 평생 그 신분으로 살아가야 한다.

#### Multilevel Feedback Queue

---

![](https://velog.velcdn.com/images/sujipark2009/post/e85595a0-d21b-4be2-89e9-8b584e717cf6/image.png)

방금전과 마찬가지로 여러 큐가 있는데, 신분 상승/하락이 가능한 방식이다.

![](https://velog.velcdn.com/images/sujipark2009/post/a88e92f2-3dd9-4a35-90f6-09c038fad3bf/image.png)

가장 우선순위가 높은 큐는 맨 위의 큐이다.

처음에 어떤 프로세스들이던 간에 맨 윗줄에 줄을 서게 된다.

만약에 8이내에 CPU를 쓰고 나갈 수 있으면 그냥 쓰고 나가면 된다.

만약 8보다 크면, 8만큼 쓰고 다음 큐로 신분이 떨어지게 된다.

아래쪽 큐는 위의 큐가 빌 때만 아래쪽 큐에 CPU가 넘어가도록 되어있다.

이건 Multilevel Feedback Queue가 꼭 이렇게 동작한다는게 아니라, 피드백 큐의 구현한 방식중 한 예시가 이렇다는 것이다.

원래 Multilevel Feedback Queue는 큐를 여러 개 둘 수 있고, 각 큐마다 스케쥴링 알고리즘을 다르게 할 수 있다.

또한 승격/강등 기준이 있으며, 처음에 들어갈 큐를 결정하는 이런 파라미터들을 다 결정해야한다.

Multilevel Queue와의 차이점은, Multilevel Queue는 starvation이 있을 가능성이 있지만, Multilevel Feedback Queue는 Aging 방식으로 구현이 가능하다.

만약 오래 기다리면 아래쪽에 우선순위를 더 줄 수도 있고, 무조건 위가 비어야만 아래를 할 수 있고 그런건 아니다.

![](https://velog.velcdn.com/images/sujipark2009/post/dea58f29-bdef-4a3b-ac50-29bc91126b51/image.png)

구체적으로는 이런 방식을 많이 쓴다는 것이다.

위가 가장 우선순위가 높고, 아래로 갈수록 낮아지는..

할당시간 안에 처리가 안되면 아래로 내려가는 이런게 가장 대표적인 구현의 예시이다.
