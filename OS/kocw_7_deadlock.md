# Deadlock(교착상태)

## Deadlock이란

**Deadlock**

- 일련의 프로세스들이 서로가 가진 자원을 기다리며 block된 상태

**Resource**

- 자원으로 하드웨어, 소프트웨어 등을 포함하는 개념
- 프로세스가 자원을 사용하는 절차
    
    Request, Allocate, Use, Release
    

**Deadlock 예시**

1. 프로세스 P1, P2와 tape drive 2개 존재
    
    이때, 각각이 하나의 tape drive를 보유한 채 다른 하나를 기다리는 상황
    
2. Binary semaphores A and B
    
    P0 → P(A); P(B);
    
    P1 → P(B); P(A);
    

## Deadlock 발생의 4가지 조건

1. 상호 배제(Mutual exclusion)
    
    매 순간 하나의 프로세스만이 자원을 사용할 수 있음
    
2. 비선점(No preemption)
    
    프로세스는 자원을 스스로 내어놓을 뿐 강제로 빼앗기지 않음
    
3. 보유 대기(Hold and wait)
    
    자원을 가진 프로세스가 다른 자원을 기다릴 때 보유 자원을 놓지 않고 계속 가지고 있음
    
4. 순환 대기(Circular wait)
    
    자원을 기다리는 프로세스간에 사이클이 형성되어야 함
    

## 자원 할당 그래프(Resource-Allocation Graph)

Deadlock이 발생하는지 확인할 때 사용하는 그래프

**자원 할당 그래프 예시**

<img width="500" alt="os_2_3" src="https://github.com/KimMinJeong05/TIL/assets/55101567/3f595c0f-a447-4daa-98f7-28dceb2baafb">

- 그래프에 cycle이 없으면 Deadlock이 아님
- 그래프에 cycle이 있으면
    - 자원 당 인스턴스가 하나씩만 있으면 Deadlock
    - 자원에 여러개의 인스턴스가 있다면, Deadlock일수도있고 아닐수도 있음

## Deadlock의 처리 방법

1. Deadlock Prevention(방지)
    
    자원 할당 시 Deadlock의 4가지 필요 조건 중 어느 하나가 만족되지 않도록하는 것
    
    → Utilization 저하, throughput 감소, starvation 문제 발생
    
    - 상호 배제(Mutual exclusion)
        
        공유해서는 안되는 자원의 경우, 반드시 성립해야함
        
    - 비선점(No preemption)
        
        자원을 기다려야하는 경우 이미 보유한 자원이 선점되고, 모든 필요한 자원을 얻을 수 있을 때 그 프로세스를 시작함
        
        State를 쉽게 save하고 restore할 수 있는 자원에서 주로 사용(ex. CPU, memory)
        
    - 보유 대기(Hold and wait)
        
        프로세스가 자원을 요청할 때 다른 어떤 자원도 가지고 있지 않아야 함
        
        방법1. 프로세스 시작 시 모든 필요한 자원을 할당받게 하는 방법
        
        방법2. 자원이 필요할 경우 보유 자원을 모두 놓고 다시 요청
        
    - 순환 대기(Circular wait)
        
        모든 자원 유형에 할당 순서를 정하여 정해진 순서대로만 자원 할당
        
        ex. R3을 보유 중에 R1을 할당 받으려면 우선 R3을 release해야함
        
2. Deadlock Avoidance(방지)
    
    자원 요청에 대한 부가적인 정보를 이용해서 deadlock의 가능성이 없는 경우에만 자원 할당
    
    평생 사용할 자원들을 미리 declare해서 deadlock 피함
    
    가용 자원만으로 Safe Sequence가 있어서 모든 프로세스가 끝까지 수행될 수 있도록 하는 것
    
    - safe state
        
        시스템 내의 프로세스들에 대한 safe sequence가 존재하는 상태
        
    - safe sequence
        
        프로세스의 sequence<P1, P2, … ,Pn>이 safe하려면 Pi의 자원 요청이 [가용 자원 + 모든 P의 보유자원]에 의해 충족되어야 함
        
    - 자원 당 1개의 인스턴스만 있는 경우, 자원 할당 그래프 사용
        
        Deadlock 위험성(unsafe)있다면, 자원이 있어도 할당해주지 않음
        
        <img width="600" alt="os_2_3" src="https://github.com/KimMinJeong05/TIL/assets/55101567/44d4c00f-0c02-4912-9dcf-adbe34417072">
        
        → 지금은 아니여도 한 번은 사용하는 자원은 claim edge(점선)
        
        → 자원을 요청하면 request edge로 바뀜(실선)
        
        → release되면 assignment edge는 다시 claim edge로 바뀜
        
    - 자원 당 여러개의 인스턴스가 있는 경우, Banker’s Algorithm 사용
        
        <img width="500" alt="os_2_3" src="https://github.com/KimMinJeong05/TIL/assets/55101567/9eb6acf4-d7db-4cee-8a37-bb57bf1f33ac">
        
        → 자원 요청 시, Available에서 요청한 값을 뺀 것보다 Need(최악의 경우)가 더 크면 Deadlock 가능성 생김
        
        → Deadlock 가능성이 생길만한 요청에는 자원을 할당하지 않음
        
        → 아래 sequence는 P1의 최악의 경우를 Available이 커버 가능. P1 자원 해제하고 P3 확인. 하는 식으로 진행되어 모두 가능하면 safe state
        
3. Deadlock Detection and recovery(처리)
    
    Deadlock 발생은 허용하되 그에 대한 detection 루틴을 두어 deadlock 발견시 recover
    
    - Detection
        - 자원 당 1개의 인스턴스만 있는 경우, 자원 할당 그래프 사용
            
            <img width="400" alt="os_2_3" src="https://github.com/KimMinJeong05/TIL/assets/55101567/fcefedcd-872a-4ca5-9c9c-a12185ed47e2">
            
            → 사이클이 존재하는지는 O(n^2) 걸림
            
        - 자원 당 여러개의 인스턴스가 있는 경우, Wait-for graph 알고리즘 사용
            
            <img width="400" alt="os_2_3" src="https://github.com/KimMinJeong05/TIL/assets/55101567/c1545207-c800-46f5-a301-fb6e0250ed18">
            
            → sequence는 P0가 끝나면 P0를 반납하고 P2를 Request하는 식으로 가정
            
    - Recovery
        - Process termination(프로세스 kill)
            - Deadlock에 연관된 프로세스 모두 kill
            - Deadlock이 없어질 때까지 Deadlock에 연관된 프로세스를 하나씩 kill
        - Resource Preemption(자원 뺏기)
            - 비용을 최소화할 victim 선정
                
                safe state로 rollback하여 process를 restart
                
                → Starvation 문제 발생 가능 → 비용 최소만 고려하는 것이 아니라 cost factor에 rollback 횟수도 같이 고려
                
4. Deadlock Ignorance(처리)
    
    Deadlock을 시스템이 책임지지 않음.
    
    Deadlock 예방 및 detection 모두 overhead가 커서 아무것도 하지 않음.
    
    비정상적으로 작동하는 것을 사람이 직접 프로세스 kill하는 방법으로 대처
    
    대부분의 운영체제가 채택