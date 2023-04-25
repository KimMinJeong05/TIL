# CPU 스케줄링

- 여러 종류의 job(process)이 섞여있기 때문에 CPU 스케줄링이 필요
    - Interactive job에게 적절한 response 제공 요망
    - CPU와 IO 장치 등 시스템 자원을 골고루 효율적으로 사용
- 프로세스의 특성 분류
    
    <img width="200" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234287956-7fffbf34-7c25-49b2-91a3-f9bff84c0897.png">
    
    1. IO-bound process
        - CPU를 잡고 계산하는 시간보다 IO에 많은 시간이 필요한 job(many short CPU bursts)
    2. CPU-bound process
        - 계산 위주의 job(few very long CPU bursts)

## CPU Scheduler & Dispatcher

- CPU Scheduler
    - Ready 상태의 프로세스 중 어떤 프로세스에게 CPU를 할당할지 결정하는 OS코드
- Dispatcher
    - CPU의 제어권을 CPU scheduler에 의해 선택된 프로세스가 받아 수행할 수 있도록 환경설정하는 OS코드 = 문맥 교환

</br>

- CPU 스케줄링이 필요한 경우는 프로세스에게 다음과 같은 상태 변화가 있는 경우 2가지 경우
    1. 비선점형 nonpreemptive(자진반납)
        1. Running → Blocked (ex. IO 요청)
        2. Terminate (ex. 프로세스 종료)
    2. 선점형 preemptive(강제로 빼앗음, 현재 사용)
        1. Running → Ready (ex. timer interrupt)
        2. Blocked → Ready (ex. IO 완료 후 interrupt)

## Scheduling Criteria(스케줄링 성능 평가)

1. 시스템 관점의 성능 지표
    1. CPU utilization(이용률)
        - CPU가 일을 한 시간/전체시간 (가능한 CPU를 많이 사용해야함)
    2. Throughput(처리량)
        - 주어진 시간 동안 Ready Queue에서 기다리고 있는 프로세스 중 몇 개가 원하는 만큼의 CPU를 사용하고 이번 CPU 버스트를 끝내어 준비 큐를 떠났는지 측정
2. 사용자/프로세스 관점의 성능 지표
    1. Turnaround time(소요시간, 반환시간)
        - 프로세스가 CPU를 기다리는 시간 + CPU 버스트가 완료될 될때까지 사용한 시간(IO에게) = 총 시간
    2. Waiting time(대기시간)
        - CPU 버스트 기간 중 CPU를 기다리는 시간의 합
    3. Response time(응답시간)
        - ready Queue에 들어가 처음으로 CPU를 얻을 때까지 기다린 시간

## 스케줄링 알고리즘

### FCFS(First-Come First-served, 선입선출 알고리즘)

- 먼저 도착한 순서대로 처리
- 단점
    1. Nonpreemptive 
        
        → Convoy effect: 짧은 프로세스들이 긴 프로세스를 다 기다려야하는 것
        
        → 평균 대기시간(average waiting time)이 길어질 수 있음
        

### SJF(Shortest-Job First, 최단작업 우선 알고리즘)

- (CPU burst time)이 가장 짧은 프로세스에게 제일 먼저 CPU를 할당
- minimum average waiting time을 보장함.(Preemptive)
- 2가지 버전
    1. Nonpreemptive
        - 일단 CPU를 잡으면 이번 CPU burst가 완료될 때까지 CPU를 선점당하지 않음
    2. Preemptive
        - 현재 수행중인 프로세스의 남은 burst time보다 더 짧은 CPU burst time을 가지는 새로운 프로세스가 도착하면 CPU 빼앗김 → SRTF(Shortest-Remaining-Time-First)
- 단점
    1. Starvation(기아 현상): CPU 사용시간이 긴 프로세스는 영원히 실행되지 않을 수 있음
    2. CPU 사용 시간을 미리 알 수 없음 → 과거의 사용으로 예측만 가능
        1. exponential averaging 수식으로 추정

### Priority Scheduling(우선순위 스케줄링)

- 우선순위(number)가 가장 높은 순으로 CPU 할당. smallest number = highest priority
- 2가지 버전
    1. Nonpreemptive
        - 일단 CPU를 잡으면 이번 CPU burst가 완료될 때까지 CPU를 선점당하지 않음
    2. Preemptive
        - 지금 CPU보다 우선순위가 더 높은 프로세스가 오면 CPU 빼앗김
- 단점
    1. Starvation(기아 현상): 우선순위가 낮은 프로세스가 영원히 CPU를 얻을 수 없음
- 단점 해결 방법
    1. Aging(노화): 오래 기다릴수록 우선순위를 높여주는 방법

### RR Scheduling(라운드 로빈 스케줄링)

- 각 프로세스는 동일한 크기의 할당 시간(time quantum)을 가짐
- 할당 시간이 지나면, 프로세스는 선점당하고 Ready Queue의 제일 뒤에 다시 줄을 섬
- 장점
    1. 응답시간이 빨라짐
        
        n개 프로세스, q time unit → 어떤 프로세스도 (n-1)*q time unit 이상 기다리지 않음
        
- 성능
    1. 할당 시간이 아주 크면 → FCFS
    2. 할당 시간이 아주 작으면 → context switch 오버헤드가 커짐. 평균 대기시간과 소요시간이 FCFS보다 비효율적임,
    
    → 적당한 할당시간이 좋음(10~100 ms)
    

- 일반적으로 SFJ보다 average turnaround time이 길지만 response time은 더 짧다

</br>

### Multi-level Queue

- Ready Queue를 여러 개로 분할해 관리하는 스케줄링 기법
    
    <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234288343-593e163b-ddb9-4a2a-9b54-87edb08c33fc.png">
    
    1. foreground(interactive한 프로세스들)
    2. background(batch - no human interactive한 프로세스들)
- 각 큐는 독립적인 스케줄링 알고리즘을 가짐
    1. foreground - RR (응답이 빠르게)
    2. background - FCFS(context switch가 덜 일어나도록)
- 한 프로세스는 정해진 큐에만 들어갈 수 있음. 다른 큐에 들어가지 못함
- 큐에 대한 스케줄링
    1. 고정 우선순위 방식(Fixed priority scheduling)
        - 무조건 우선순위가 높은 큐부터 CPU 할당
            
            → Starvation이 일어날 가능성있음
            
    2. 타임 슬라이스 방식(Time slice)
        - 각 큐에 CPU time을 적절한 비율로 할당

### Multilevel Feedback Queue

- CPU를 기다리는 프로세스를 여러 큐에 줄을 세우는데, 프로세스가 다른 큐로 이동 가능
- Aging을 구현할 수 있음
- 프로세스의 실행 시간이 해당 큐의 할당 시간보다 클 경우, 다음 큐로 넘어가는 방식
    
    <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234288653-166b0acf-03e9-4548-8f33-561f7eeb98ba.png">
    
    → 실행 시간이 짧은 프로세스를 우선적으로 처리
    
    → 프로세스의 실행 시간을 예측할 필요가 없음. 처음엔 짧게 주고, 끝내지 못하면 다음 큐로 넘김
    
</br>

### Multiple-Processor Scheduling

- CPU가 여러 개인 경우
1. Homogeneous processor인 경우
    - Queue에 한줄로 세워 CPU들에게 꺼내줌
    - 반드시 특정 프로세서에서 수행되어야 하는 프로세스가 있는 경우, 문제가 복잡해짐
2. Load sharing
    - 일부 프로세서에 job이 몰리지 않도록 부하를 적절히 공유하는 메커니즘 필요
    - 별개의 큐를 두는 방법 OR 공동 큐를 사용하는 방법
3. Symmetric Multiprocessing(SMP)
    - 모든 CPU가 대등해서, 각 프로세서가 각자 알아서 스케줄링 결정
4. Asymmetric Multiprocessing
    - 하나의 프로세서가 시스템 데이터의 접근과 공유를 책임지고 나머지 프로세서는 거기에 따름

### Real-Time Scheduling

- 정해진 시간내에 반드시 실행되어야하는 것
- 미리 스케줄링해서 deadline에 끝낼 수 있도록 함
1. Hard real-time systems
    - 정해진 시간 안에 반드시 끝내도록 스케줄링 함
2. Soft real-time computing
    - 일반 프로세스에 비해 높은 priority를 갖도록 해야 함

### Thread Scheduling

1. Local Scheduling
    - User Level Thread의 경우 사용자 수준의 thread library에 의해 어떤 thread를 스케줄할지 결정
        - 사용자 process 내부에서 직접 thread를 관리. OS는 thread의 존재를 모름
2. Global Scheduling
    - Kernel Level Thread의 경우 일반 프로세스와 마찬가지로 커널의 단기 스케줄러가 어떤 thread를 스케줄할지 결정
        - OS가 thread의 존재를 앎. 프로세스 스케줄링하듯이 스레드 스케줄링 함

## Algorithm Evaluation

1. Queueing models
    - 확률 분포로 주어지는 arrival rate(도착률)와 service rate(처리율)등을 통해 각종 performance index값을 계산
2. Implementation(구현) & Measurement(성능 측정)
    - 실제 시트셈에 알고리즘을 구현하여 실제 작업에 대해서 성능을 측정 비교
3. Simulation(모의 실험)
    - 알고리즘을 모의 프로그램으로 작성 후 trace(input 데이터)를 입력으로 하여 결과 비교

</br>

### 레퍼런스

---

- 반효경, 운영체제, KOCW, [http://www.kocw.net/home/search/kemView.do?kemId=1046323](http://www.kocw.net/home/search/kemView.do?kemId=1046323)
- 반효경, 운영체제와 정보기술의 원리