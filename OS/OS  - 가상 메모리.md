### 배경지식

#### Demand Paging

---

![](https://velog.velcdn.com/images/sujipark2009/post/5369e2ea-b00d-4eb2-8629-1338352f4ce5/image.png)

물리적인 메모리의 주소변환은 OS가 관여하지 않는다고 했었다.

가상메모리 기법은, 전적으로 **OS가 관여하는 기법**이다.

여기서부터는 페이징 기법을 사용한다고 가정한다.

**Demand Paging**은, 페이지가 요청이 되었을 때 페이지를 메모리에 올려놓는 기법이다.

이것으로 인해 I/O의 양을 줄일 수 있다.
좋은 프로그램일수록 예외처리를 위한 코드가 많이 있는데, 그런 코드들은 가끔씩만 사용이 된다.

그런 코드들도 메모리에 올려놓게 된다면, 메모리를 많이 잡아먹게 된다.

만약, Demand Paging 기법을 사용하면 필요한것만 메모리에 올리기 때문에 I/O의 양이 감소하게되고, 물리적 메모리를 사용하는 양이 감소하게 된다.

페이징 기법에서 페이지 테이블 엔트리마다 valid/invalid bit이 있다고 했었다.

![](https://velog.velcdn.com/images/sujipark2009/post/c56a601c-e9f0-42cf-89bf-56ada67ea984/image.png)

지금 당장 필요한 페이지는 demand paging에 의해 메모리에 올라가있고, 그렇지 않은것은 backing store에 저장되어있다.

store에 저장되어있는 나머지 페이지에 대해서도 bit가 invalid로 표시되어있다.
또한, 사용이 안되는 페이지에 대해서도 invalid로 표시되어 있다.

실제 사용되는 페이지와 그렇지 않은 페이지가 있는데, 더 넉넉한 주소공간을 지원해주기 때문에 실제 사용이 안되는 G,H에 대해서도 invalid로 표시되어있다.

CPU가 논리주소를 주고 메모리 몇 번지를 읽겠다! 하고 왔는데 invalid라면?
이 말은, 이 페이지가 메모리에 없다는 말이다.
그 페이지를 먼저 Disk에서 올려야하고 이것은 I/O 작업이므로 사용자 프로세스가 할 수 없는 작업이다.

이것은 **OS가 해야한다**

이렇게 Page가 없는 상황에 `Page fault`라는 Trap이 걸리게 되고, 이때 OS가 CPU를 받아 해당 page를 메모리에 올리게 된다.

#### Page Fault

---

![](https://velog.velcdn.com/images/sujipark2009/post/e7164e3e-3b71-4963-b1d6-176403852049/image.png)

invalid page를 접근하면, MMU(주소변환을 위한 HW)가 Trap을 발생시킨다.
그러면 CPU가 OS에게 넘어가게되고 OS의 Page fault를 처리하는 코드가 다음의 순서로 실행이 된다.

1. 유효한 주소가 맞는지(이 프로세스가 접근할 수 있는 주소가 맞는지)

2. 빈 페이지 하나를 획득해야하는데, 없으면 하나를 쫓아내야 한다.

3. Disk에서 메모리로 그 페이지를 올리게 된다.
   3-1. 이 작업은 느린 작업으로, Disk I/O를 하게 되면 Block상태가 되어 CPU를 뺏기게 된다.
   3-2. Page table Entry에다가 valid로 표시한다.
   3-3. Ready Queue에 해당 Process를 넣는다.
4. 이 프로세스가 CPU를 받으면 Running 상태가 된다.

![](https://velog.velcdn.com/images/sujipark2009/post/83516226-7129-4647-97ee-4620e57e47fd/image.png)

그림을 보면, 주소 변환을 하려고 페이지 테이블을 봤더니 `invalid`라고 표시되어있는 상황이다.

페이지가 현재 메모리에 올라와있지 않다는 의미이니, trap이 걸려서 OS에게 CPU가 넘어가게 된다.

OS는 Backing store에 있는 page를 물리적 메모리로 올려놓는데 이때, 빈 페이지 프레임이 없으면 하나를 쫓아내게 된다.

그리고 페이지를 메모리에 올리고 Page를 valid로 변경한다.

그리고 다시 CPU를 얻어서 주소변환을 하면 valid로 되어있으므로, 실제 물리적 메모리에 접근할 수 있게 된다.

![](https://velog.velcdn.com/images/sujipark2009/post/971ce236-8ba4-4c48-9367-df5bdef42759/image.png)

Page fault시 Disk에 접근하는것은 매우 느리다.

Fault가 발생하면 OS에게 CPU가 넘어가서 Fault를 처리해야하고, 그 과정에서 빈 페이지가 없으면 쫓아내야하고, valid로 표시해야하고, CPU를 얻으면 running하는 이런 overhead가 존재한다.

![](https://velog.velcdn.com/images/sujipark2009/post/85302128-8f0b-4875-93a9-13be2bed15c5/image.png)

빈 프레임이 없으면 쫓아내야한다고 했는데, 그것을 `Page Replacement`라고 한다.

이것은 OS가 하는 일로, 어떤 페이지를 메모리에 쫓아내고 새 페이지를 올릴지를 결정한다.
이것을 위한 알고리즘을 `Replacement Algorithm`이라고 한다.

페이지를 쫓아냈더니 다시 그 페이지를 필요로 한다면, 엄청난 손해를 입기 때문에 이 알고리즘은 **가급적 Page fault rate가 낮아지도록** 동작해야한다.

![](https://velog.velcdn.com/images/sujipark2009/post/4195b731-8847-4c4f-83e1-b8ce1a4a13a6/image.png)

그림처럼 내려갈 페이지(victim)을 결정했으면 디스크로 쫓아내야한다.

그런데, 이 페이지가 디스크에서 메모리로 올라온 이후로 내용이 변경되었다면 그냥 쫓아내는게 아니라 변경된 내용을 Backing store에 써줘야한다.

변경된게 없다면 그냥 지우기만 하면 된다.

![](https://velog.velcdn.com/images/sujipark2009/post/196f2527-1f6a-4a86-a642-da8fd6e2f774/image.png)

어떤 알고리즘이 가장 좋은 알고리즘일까?

**Optimal Algorithm**은 미래에 어떤 페이지가 올 지 알고있다는 가정하에 사용할 수 있는 알고리즘이다.

현실성이 없는 알고리즘으로, 가장 미래에 참조될 페이지를 쫓아내는 방식이다.

이 경우, 6번의 Page fault가 발생하는데, 이게 최적이기에 이것보다 Fault수를 줄일 수는 없다.
다른 알고리즘의 성능에 대한 `Upper Bound`를 제공한다.

아무리 좋은 알고리즘을 만들어도, 이것보다 좋을수는 없다는 것이다.

여기서부터의 알고리즘은, 실제 시스템에서 사용가능하고 미래를 모른다는 가정하에 사용할 수 있는 알고리즘이다.

![](https://velog.velcdn.com/images/sujipark2009/post/9411ff46-0bad-4cac-aab5-98371474fb37/image.png)

FIFO(First in First out)

말 그대로 먼저 들어온 것을 내쫓는 방식이다.

이 알고리즘은 특이한 성질이 있는데, 메모리 프레임의 수를 늘려주면 성능이 더 좋아져야 하는데 오히려 Page fault가 늘어나는 현상이 있다.

#### LRU(Least Recently Used)

---

![](https://velog.velcdn.com/images/sujipark2009/post/ccf0c943-a2a0-470b-b18f-96eb49c384d6/image.png)

가장 널리 사용되는 알고리즘이다.
**가장 오래된 페이지를 쫓아내는 알고리즘**이다.

#### LFU(Least Frequently Used)

---

![](https://velog.velcdn.com/images/sujipark2009/post/07849783-65df-47e7-966f-e1c49e94d3f3/image.png)

**가장 덜 빈번하게 사용된 페이지를 쫓아내는 방식**이다.

과거에 참조횟수가 많았던 페이지는 미래에도 많을 수 있으므로 쫓아내지 말자는 것이다.

동률인 경우, 임의로 선정할 수 있고 성능향상을 생각한다면 가장 참조시점이 오래된 page를 지우도록 구현할 수 있다.

![](https://velog.velcdn.com/images/sujipark2009/post/0fc8005d-c193-4ca9-9950-7a805c707b16/image.png)

Reference String이 1,1,1,1,2,2,3,3,2,4,5라고 해보자.

그리고 4개의 페이지 프레임이 있다고 해보자.

저 현재시각시점에 5번페이지를 참조해야하는 상황이라고 해보자.

LRU의 경우, 1번 페이지를 쫓아내게 된다.

- 마지막 참조 시점만 본다는 약점이 있다.
- 과거에 많이 참조했었다는 것을 반영하지 못한다.

LFU의 경우, 4번 페이지를 쫓아내게 된다.

- 비록 4번의 횟수가 1번이지만, 이제 막 참조하려는 상황일수도 있다.

![](https://velog.velcdn.com/images/sujipark2009/post/52478708-cd23-4764-8b10-6f2832ade44e/image.png)

LRU 알고리즘은 메모리 안에있는 페이지들을 시간순서에따라 줄 세우기를 한다.
일종의 **LinkedList**의 형태이다.
메모리 내에서 재참조 되거나, 새로운 페이지가 들어오면 가장 아래로 보내면 된다.

쫓아낼때는 가장 위에있는 것을 쫓아내면 된다.
이렇게 하면 O(1)의 복잡도로 구현이 가능하다.

LFU 알고리즘은 비슷하게 한 줄로 줄세우기를 하는데
가장 참조횟수가 적은것이 위에오고 많은것이 아래에 위치한다.

사실 LFU는 한 줄로 줄세우기를 할 수 없는데, 지금 참조되었다는건 참조횟수가 1이 늘어났다는 의미이지 그렇다고 해서 가장 밑으로 내려갈 수 있는게 아니라, 비교를 해서 아래로 내려가면서 위치를 찾아야하기 때문이다.

최악의 경우 O(N)이 될 수 있다.

![](https://velog.velcdn.com/images/sujipark2009/post/0714d9c0-4d27-454c-8e1b-650cdabcc93b/image.png)

그래서 사실 Heap 자료구조를 사용해서 구현하게 된다.

맨 위는 참조횟수가 가장 적은 페이지, 밑으로 갈수록 자식이 부모보다 참조횟수가 더 많은 페이지들이 있다. 일종의 최소힙이라고 할 수 있겠다.

저 높이가 logN이 되기에, 비교횟수가 많아봤자 logN이 된다.

그런데, 실제 페이징 시스템에서는 LRU,LFU는 사용할 수 없다.

왜 그럴까?

![](https://velog.velcdn.com/images/sujipark2009/post/f7780375-d7f1-4b40-8998-e427370d5eac/image.png)

Process A가 지금 CPU를 가지면서 running을 하고 있다고 해보자.

그림은 A라는 프로세스의 논리적 메모리, 페이지 테이블, 물리적 메모리, 그리고 backing store이다.

이 프로그램이 메모리 참조를 하기위해서 주소변환을 했더니, 그 내용이 이미 물리적 메모리에 올라와있다면 그 페이지를 바로 CPU가 가지고 갈 것이다.

이렇게 CPU가 메모리에 올라와있는 것을 그냥 가져가게되면, 중간에 OS가 개입하는게 전혀 없어진다.

다시말해서, **이 페이지를 언제 사용했는지, 그 사실을 OS가 알 수 없다**는 것이다.

LRU 알고리즘을 운영하려면, 그 페이지들이 사용되는 시점을 알아야하는데, 저렇게 바로 가져가게 된다면 OS는 알 방법이 없다.

반대로 A라는 프로그램이 주소변환을 해본 결과 페이지가 없다면, 그런 경우에는 Page fault가 발생하고 CPU제어권이 OS에게 넘어갈 것이다.

OS가 Backing Store에 있는 페이지를 올려놓게 되고 그때 언제 올린지 시간을 알 수 있게 되므로, 그 페이지를 LRU 하단에 매달아 놓을 수 있게된다.

그래서 페이징 시스템에서 OS는 **페이지 정보를 반만 알고있다**고 할 수 있다.
처음 사용이 될 시점에만 기록할 수 있기 때문이다.

그래서 **실제로 페이징 시스템에서는 LRU,LFU 알고리즘을 사용할 수 없기 때문에, LRU를 근사시킨 알고리즘인 `Clock Algorithm`**을 사용한다.

![](https://velog.velcdn.com/images/sujipark2009/post/ca9c4ab6-9753-40c5-b74d-5e80aecb694b/image.png)

사각형 하나하나는 메모리 안의 페이지를 의미한다.

Replacement algorithm이므로, 이 페이지들 중 하나를 쫓아내야하는 상황이다.

각 페이지마다 0또는 1이 적혀있는데, 이 bit를 `referece bit` 혹은 `access bit`이라고 한다.

1이라고 한다면, 최근에 사용된 것이고 0이면 최근에 사용이 안된 페이지라는 의미이다.
LRU의 경우, 가장 오래된 페이지를 쫓아낸다고 했었다.

하지만 모든 페이지들의 사용시점을 OS가 다 알수는 없기에 bit하나만 가지고 쫓아낼 페이지를 결정하게 된다.

그래서 0으로 표시된 페이지를 쫓아내게 된다.

0으로 표현되었다고 해서 가장 오래된 페이지는 아니지만, 적어도 최근에 사용이 안된 페이지는 맞다.

이걸 구현하는 방법을 알아보자.

Reference bit을 1로 만드는것은 HW가 하는 일이다.
HW가 CPU가 주소변환을 해서 그 페이지를 가져갔다! 하면 1로 두는 것이다.

근데 OS가 이 중 하나를 쫓아내야겠다고 한다면 시계바늘이 돌면서 bit가 1이면 0으로 바꾸고 다음 칸으로 화살을 이동하게 된다.

그러다가 만약0이라면 쫓아내게 되는 것이다.

bit를 1로 바꾸는건 HW가, 0으로 바꾸는건 OS가 하게 된다.

이렇게 하면 bit가 1이라는 의미는 시계바늘이 한바퀴 돌아오는 동안 적어도 그 페이지가 한번은 사용되었다는 의미이다.

0은 한 번 돌아올때까지 사용이 안되었다는 의미이므로 쫓아내게 된다.

reference bit 말고 `modified bit / dirty bit`도 있다.

이 메모리 페이지가 사용되는건 CPU의 read 혹은 write가 있는데

페이지가 올라온 다음에 CPU가 접근을 하면 HW가 reference bit을 1로 바꾸어 주고 CPU가 write를 해서 수정하게 된다면, 그 페이지에 대한 modified bit을 1로 바꾸게 된다.

이 modified bit을 왜 쓰냐면, 저 페이지가 올라온 이후로 수정이 되었으면 쫓아낼 때 그냥 없애는게 아니라 disk에 써줘야하기 때문에 그것을 알기위한 목적이다.

#### Page Frame Allocation

---

![](https://velog.velcdn.com/images/sujipark2009/post/f3c7831f-4235-4b7f-bb3f-8efd17bc384d/image.png)

앞서 설명한 알고리즘은 어떤 프로그램에 의해서 사용되는 페이지인지는 전혀 고려하지 않았다.

물리적 메모리에는 A라는 프로세스의 페이지 / B라는 프로세스의 페이지.. 이렇게 다 올라가있다.

쫓아낼 때는 LRU 알고리즘을 사용한다면 그 중 가장 오래된 페이지를 쫓아내게 되는데
프로세스 A가 사용한다고 해서 쫓아내고, B가 쓰는거라 봐주는 등 그런게 없었다.

여기서 말하는 Allocation 이라는 의미는, 프로세스 A에게 이만큼, B에게 이만큼을 할당하겠다는 의밍이다.

할당 이야기를 하는 이유는, 프로그램이 원활하게 실행되려면 적어도 메모리 상 페이지 프레임을 몇 개는 가지고 있어야하기 때문이다.

만약 지금 반복문을 돌고있다고 해보자. 그 반복문을 구성하는 페이지가 5개이고 5개의 페이지를 할당했다면 반복문을 1천만번을 돌아도 Page fault가 안 날 것이다.

그런데 만약 3개만 줬다면, 반복문을 도는 동안 계속 Page fault가 날 것이다.

그것은 대단히 비효율적이기 때문에, **각 프로세스 당 최소로 가져야하는 프레임**이 있다.

그래서 할당이 필요하다는 것이다.

프로그램을 구성하는 **code,data,stack**이 있는데, code만 올라가있다고 해서 fault가 안나는것이 아니고, data도 올라가 있어야 한다.

할당 방식은, `Equal Allocation`과 ,`Preportional Allocation`,`Priority Allocation`이 있는데

Equal의 경우, 모든 프로세스에 공평한 개수를 할당하는 것이고

Preportional은 프로세스 크기에 비례해서 할당,

Priority의 경우 CPU 우선순위가 높은 프로세스에게 더 많은 페이지 프레임을 할당하는 것이다.

#### Thrashing

---

![](https://velog.velcdn.com/images/sujipark2009/post/fc41f8f0-acbe-4a9c-810c-be519dfe12ac/image.png)

x축은 메모리에 올라가있는 프로그램의 수
y축은 CPU사용률(CPU가 단위시간당 일한 비율) 이다.

메모리에 프로그램 하나가 올라가있으면 그 프로그램이 CPU를 쓸 것이다.
그렇게 쓰다가 I/O를 하러 가버리면 CPU가 놀기때문에, CPU 사용률이 낮아진다.

이런식으로 프로세스의 수를 증가시키면 사용률이 조금씩 증가하다가, 어느 순간 떨어지는데 이걸 `Thrashing` 이라고 한다.

메모리에 너무 많은 프로그램을 동시에 올려놓읜, 프로그램이 원활하게 운용되기 위해 필요한 최소한의 메모리도 얻지 못한 상황인 것이다.

그래서 CPU를 줘도 계속 Page fault가 나는 상황이다.
CPU는 놀고있고, Page fault를 처리하느라 바쁜 상황이다.

![](https://velog.velcdn.com/images/sujipark2009/post/a44efe1e-8c27-4a60-82ad-5942e2bc661a/image.png)

Thrashing이 발생해서 CPU가 놀고있으면, OS는 프로그램을 더 넣어야겠다고 판단하고 결국 시스템이 더 안좋아지게 된다.

CPU는 일을 못하고, I/O만 하게되는 상황이 발생한다.

그래서 thrashing을 막아야하는데, 이것을 막는 방법은 그 프로그램이 필요로 하는 최소한의 메모리는 보장해주는 것이다.

![](https://velog.velcdn.com/images/sujipark2009/post/bd5772f9-45eb-4a43-ba61-9beff7a0fe08/image.png)

프로세스는 특정 시간동안에 특정 메모리를 집중적으로 사용하는 경향이 있다.

예를들어 어떤 함수를 실행하고있다하면, 그 함수가 있는 페이지가 집중적으로 사용이 될 것이다.

그런 집중적으로 사용되는 page들의 집합을 Locality set이라 하고 이걸 보장해 줘야 원활하게 프로그램이 동작한다는 것

그런데, Working-set Model에서는 그러한 locality set의 집합을 working set이라고 부른다. 사실 둘이 같은말임..

working set을 이용한 메모리 관리 알고리즘은, 저 working set에 포함된 페이지들을 무조건 메모리에 보장하도록 작동한다.

만약 5개가 필요한데 빈 공간이 3개밖에없다?

그럼 그냥 메모리를 다 반납하고 swap out 되어라(suspended)하는 식으로 작동한다.
나중에 메모리가 좀 남아돌때 다시 제공받게 된다.

![](https://velog.velcdn.com/images/sujipark2009/post/3248f5a3-7575-438d-a758-9950fb75e993/image.png)

지금 보면 시간순서에따라 page reference string이 있는데

이건 특정 프로세스의 페이지이다.

그래서 working set이 뭔지는 정확하게 모르지만, 과거를 통해 추정을 한다.

현재 시점을 기준으로 working set을 결정하게 된다.

델타만큼의 시간동안 사용된 페이지를 유지하게된다.

대충.. 너무 깊게는 .. 알기싫네...

Working set은 Global 과 Local replacement를 적절히 섞은..것이라고 볼 수 있다.

![](https://velog.velcdn.com/images/sujipark2009/post/de3b7e75-333a-4da4-abf8-8b0e2ed33526/image.png)

보통 32bit 시스템에서는 4KB의 페이지 크기를 사용하는데

이제 64bit 주소체계를 쓰면 어떻게 해야할까?

메모리가 커지면서 메모리를 쪼개쓰는 페이지크기도 더 커지는 추세로 가게 된다.

그러면 페이지 크기를 키우거나 줄이거나 했을 때 전체 시스템에 어떤 영향을 미칠까?

페이지 크기를 줄이면?

기존에

![](https://velog.velcdn.com/images/sujipark2009/post/6c847373-8853-4e90-adbe-ca48dd7f7dff/image.png)

8개로 쪼개던걸 100개로 쪼개면?..

페이지 테이블이 그만큼 엔트리가 늘어날것이다.

대신에 물리적 메모리도 작게 썰어버리니까 불필요한 공간은 사라질것임(내부 단편화가 줄어들듯)

대신에 잘게 쪼게면 단점이 페이지 테이블이 더 늘어나야 한다는 문제가 있고

보통 페이지가 사용이 되면, 어떤 위치가 사용되면 그 인접한 위치가 사용될 확률이 높은데,.. 그런 관점에서 페이지 폴트가 많이 날 것이다.

그말이 Disk transfer의 효율성이 감소한다는 것이다.

기존에 크게 읽었으면 한번에 읽어올걸 작게 쪼개놔서 여러개를 읽어야하니까..
Locality 측면에서 좀 손해다.
