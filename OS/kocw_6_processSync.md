# 프로세스 동기화

## Process Synchronization

= Concurrency Control (병행 제어)

- 공유 데이터의 동시 접근은 데이터의 불일치 문제(inconsistency)를 발생시킬 수 있음
- 일관성 유지를 위해서는 협력 프로세스 간의 실행 순서를 정해주는 메터니즘 필요

## **Race Condition(경쟁 상태)**

- 여러 프로세스들이 동시에 데이터에 접근하는 상황에서, 어떤 순서로 데이터에 접근하느냐에 따라 결과 값이 달라질 수 있는 상황
    - 예시
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234915380-77cd0884-d445-4d3e-b1aa-d39c4ddd9c82.png">
        
        → S-Box를 공유하는 E-Box가 여러 개가 있는 경우 **Race Condition**의 가능성이 있음
        
        ex) Multiprocessor System, 커널 내부 데이터를 접근하는 루틴들 간
        
    
    ⇒ Race Condition을 막기 위해서는 협력 프로세스는 **동기화**되어야 함

</br>

- OS에서 Race Condition은 언제 발생하는가?
    1. kernel 수행 중 인터럽트 발생 시
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234915709-9a98f2bc-d757-4a4d-a511-5394b637f04f.png">
        
        - 의도: ++와 —연산으로 인해 원래 기본 값
        - 결과: interrupt에서의 빼기 연산이 되지 않아 ++만 반영된 값
        - 해결: 변수 값을 수정하는 동안에는 interrupt 처리를 안함. 변수가 다 수정되면 다시 interrupt를 처리.
    2. Process가 system call을 하여 kernel mode로 수행 중인데 context switch가 일어나는 경우
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234915790-0b285d28-0d91-4e54-b3a5-a9d25afe3bd9.png">
        
        - 의도: B의 ++와 A의 ++연산으로 +2인 값
        - 결과: B의 연산이 되지 않아 A의 ++연산만 반영된 +1 값
        - 해결: 커널 모드에서 수행 중일 때는 CPU를 preempt(선점)하지 않음. 커널 모드에서 사용자 모드로 돌아갈 때 preempt.
    3. Multiprocessor에서 shared memory 내의 kernel data
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234915930-6dc51726-cdcb-43ea-8c11-bd1590de960f.png">
        
        - 의도: ++와 —연산으로 인해 원래 기본 값
        - 결과: race condition으로 원하는 값이 안나올 수 있음
        - 해결:
            1. 한번에 하나의 CPU만이 커널에 들어갈 수 있게 하는 방법
            2. 커널 내부에 있는 각 공유 데이터에 접근할 때마다 그 데이터에 대한 lock/unlock을 하는 방법

## The Critical-Section(임계구역) Problem

- Critical-Section: 공유데이터를 접근하는 코드
- n개의 프로세스가 공유 데이터를 동시에 사용하기를 원하는 경우 문제 발생
- 각 프로세스의 code segment에는 공유 데이터를 접근하는 코드인 critical section이 존재
- Problem
    
    <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234916158-1532bffd-cad8-429f-b91b-3805b2a1423c.png">
    
    - 하나의 프로세스가 critical section에 있을 때, 다른 모든 프로세스는 critical section에 들어갈 수 없어야 함
        
        → Lock을 걸어 다른 프로세스가 접근하지 못하도록 함
        

## The Critical-Section Solved

- 데이터를 읽는 것과 쓰는 것을 하나의 instruction으로 처리할 수 없어서 문제가 발생

</br>

- 프로그램적 해결법의 충족 조건
    1. Mutual Exclusion(상호 배제)
        
        프로세스 Pi가 critical section 부분을 수행 중이면, 다른 모든 프로세스들은 그들의 critical section에 들어가면 안됨
        
    2. Progress
        
        아무도 critical section에 있지 않은 상태에서 critical section에 들어가고자 하는 프로세스가 있으면, critical section에 들어가게 해줌
        
    3. Bounded Waiting(유한 대기)
        
        프로세스가 critical section에 들어가려고 요청한 후부터 그 요청이 허용될 때까지, 다른 프로세스들이 critical section에 들어가는 횟수에 한계가 있어야함
        
        = 기다리는 시간이 유한해야한다. Starvation이 있으면 안됨
        
</br>

- The Critical-Section Solved 알고리즘
    1. Algorithm 1
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234916407-9f636547-c4d5-4176-880f-aeed0c067638.png">
        
        → turn: 실행할 프로세스의 번호
        
        → 0번이 먼저 실행되고 turn=1이 되므로 다음으로 1번이 실행
        
        - 문제점
            - 아무도 critical section에 없는데, 못들어가는 경우가 있음
                
                → 코드를 보면 교대로 critical section에 들어가는데, 실제로 프로세스가 접근하는건 교대가 아니라 같은 프로세스가 여러 번 접근할 수 도 있음
                
    2. Algorithm 2
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234916533-4af6ac47-17b8-4dc0-8876-775c343f9181.png">
        
        → flag: 본인이 critical section에 들어갈건지에 대한 값
        
        - 문제점
            - i가 flag를 true로 한 후 CPU를 뺐겨 j가 실행되었을 때
                
                → 둘 다 2행까지 수행 후 끊임없이 양보하는 상황 발생 가능
                
    3. Algorithm 3(Peterson’s Algorithm)
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234916624-83a16f09-9643-4ffe-91b2-f598d98f990d.png">
        
        → flag, turn 변수 모두 사용
        
        → 상대방이 critical section에 접근하고 싶어하고 상대방의 차례일때만 waiting
        
        → 충족 조건을 모두 만족함
        
        - 문제점
            - Busy Waiting(=spin lock) → 계속 CPU와 메모리를 사용하면서 waiting

</br>

- 하드웨어적으로 해결방법
    - Test & modify를 atomic하게 수행할 수 있도록 지원하는 경우 간단히 해결됨
        
        → 하드웨어적으로 읽는 것과 셋팅하는 것을 하나의 instruction(Test_and_set)으로 만들어 줌
        
        <img width="300" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234916879-0b3deb7f-37bb-485b-88d9-27fca10ae799.png">
        

## Semaphores
    
- Semaphores S
    - The Critical-Section 해결방법들을 추상화시킴  
        → 추상 자료형(논리적으로 구현하는 것)
    - integer variable
    - 공유자원을 획득하고 잃는 것을 관리(ex. lock)
    - 아래의 두가지 atomic 연산에 의해서만 접근 가능
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234917039-2e8c22be-fc56-43e6-91f2-7ac7d94ce6fb.png">
        
        1. P(S): 공유 데이터를 획득(lock)
        2. V(S): 공유 데이터를 반납(unlcok)

</br>

- Semaphores의 종류
    1. Counting semaphore
        - 도메인이 0이상인 임의의 정수값
        - 주로 resource counting에 사용
    2. Binary semaphore(**=mutex**)
        - 0 또는 1 값만 가질 수 있는 semahpore
        - 주로 mutual exclusion(lock/unlock)에 사용

</br>

- Semaphores를 이용한 The Critical-Section 해결방법
    1. Critical Section of n Processes + Semaphores
        
         <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234917451-3665b72b-d3a6-447d-af0d-155446822410.png">
        
        → busy-wait(spin lock)는 효율적이지 못함
        
        → 해결하기 위해 Block & Wakeup(=sleep lock) 방식으로 구현
        
    2. Block & Wakeup 방식
        - Sempaphore 정의
            
            <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234917580-8c2231e4-9ea0-466a-a180-e39431684bb6.png">
            
        - block과 wakeup 가정
            - block
                
                커널은 block을 호출한 프로세스를 suspend시킴
                
                이 프로세스의 PCB를 semaphore에 대한 wait queue에 넣음
                
            - wakeup(P)
                
                block된 프로세스 P를 wakeup시킴
                
                이 프로세스의 PCB를 ready queue로 옮김
                
            
            <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234917759-ddbc9fdd-8ffd-4e55-99a3-06376e6aa172.png">
            
    - Busy wait vs Block/wakeup
        - 일반적으로 Block/wakeup가 좋음
        - 하지만, Critical section의 길이가 매우 짧은 경우 Block/wakeup는 오버헤드가 커질 수 있음

## Deadlock and Starvation

- Semaphore는 2가지 문제가 생길 수 있음
    1. Deadlock
        - 둘 이상의 프로세스가 서로 상대방에 의해 충족될 수 있는 event를 무한히 기다리는 현상
            
            → 자원을 얻는 순서를 똑같이 맞춰주면 해결
            
    2. Starvation
        - 프로세스가 suspend된 이후에 해당하는 세마포어 큐에서 빠져나갈 수 없는 현상(indefinite blocking)

## Classical Problems of Synchronization

1. Bounded-Buffer Problem(Producer-Consumer Problem)
    - 상황
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234917884-16ffd66a-1146-4f07-83af-3b2698f93c53.png">
        
        →  Buffer의 크기가 유한한 환경
        
        여러 생산자 프로세스: 데이터를 만들어 넣음
        
        여러 소비자 프로세스: 데이터를 꺼내감
        
    - Shared data
        - Buffer, Buffer 조작변수(empty/full buffer의 시작 위치)
    - Synchronization variables
        1. Buffer 자체에 lock/unlock을 해야함 → Binary semaphore
        2. Buffer가 유한한 크기이기 때문에 자원이 없으면 자원이 생길때까지 프로세스가 기다려야함(남은 자원의 개수가 필요함) → Counting semaphore
    - 구현
        
        <img width="300" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234918013-460cc2fb-71e3-4e13-82f5-de68f51e95e7.png">
        
2. Readers and Writers Problem
    - 상황
        - 한 프로세스가 DB에 write중 일때, 다른 프로세스가 접근하면 안됨
        - read는 동시에 여럿이 해도 됨
    - Shared data
        - DB 자체, readcount(현재 DB에 접근 중인 Reader의 수)
    - Synchronization variables
        - mutex(공유 변수 readcount를 접근하는 코드의 mutual exlusion 보장을 위해)
        - db(Reader와 Writer가 공유 DB 자체를 올바르게 접근하게 하는 역할)
    - solution
        - Writer가 DB에 접근 허가를 아직 얻지 못한 상태 → 모든 대기중인 Reader들은 다 DB 접근 가능
        - 대기 중인 Reader가 하나도 없는 상태 → Writer는 DB 접근 허용
        - Writer가 DB에 접근 중 → Reader들은 접근 금지
        - Writer가 DB에서 빠져나가야지만 → Reader의 접근 허용
    - 문제점
        - Starvation 발생 가능
            - Reader가 계속 안놓아주면 Writer가 실행되지 않을 수 있음
    - 구현
        
        <img width="300" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234918266-7307f01f-9fec-46e6-9c52-9fa446452dfe.png">
        
3. Dining-Philosophers Problem
    - 상황
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234918429-af90125b-b207-4dac-ac94-b3cf25e15977.png">
        
        →  철학자는 생각 or 밥을 먹음
        
        밥을 먹을려면 양 옆의 젓가락을 잡아야함
        
        양옆 사람이 밥을 먹으면 나는 밥을 먹을 수 없음
        
    - 위의 solution의 문제점
        - **Deadlock** 가능성이 있음
            
            → 모든 철학자가 동시에 배가 고파져 왼쪽 젓가락을 집어버린 경우
            
    - 해결 방안
        1. 4명의 철학자만이 테이블에 동시에 앉을 수 있도록 함
        2. 젓가락을 두 개 모두 집을 수 있을 때만 젓가락을 집을 수 있게 함
        3. 비대칭 즉, 짝수(홀수) 철학자는 왼쪽(오른쪽) 젓가락부터 집도록 함
    - 구현
        
        2번 해결방안
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234918530-175dbecf-c100-45b0-8e8d-35f29eccb410.png">
        

## Monitor

- **Semaphore의 문제점**
    - 코딩하기 힘들고, 정확성의 입증이 어려움
    - 자발적 협력이 필요하고, 한번의 실수가 모든 시스템에 치명적 영향을 줌
        
        Mutual exclusion이 깨지거나 Deadlock 발생할 수 있음
        
- Monitor
    - 동시 수행중인 프로세스 사이에서 abstract data type의 안전한 공유를 보장하기 위한 high-level synchronization construct
    - 공유 데이터에 접근하기 위해선 monitor의 내부 procedure을 사용해야함. 공유 데이터도 monitor 내부에 선언되어있음
        
        <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/234918810-41b44dc6-850c-41b6-b10a-cc8b00bf490b.png">
        
    - monitor 내에서는 한번에 하나의 프로세스만 활동 가능 → **lock을 걸 필요가 없음**
    - 만약 공유 데이터에 접근했는데 monitor에 활동중인 프로세스가 있다면, queue에 들어가서 기다림
    - 프로세스가 monitor 안에서 기다릴 수 있도록 하기 위해 condition variable 사용
        - Condition Variable
            - wait과 signal 연산에 의해서만 접근 가능
            - x.wait()
                
                x.wait()을 발생한 프로세스는 다른 프로세스가 x.signal()을 발생하기 전까지 suspend됨
                
            - x.signal()
                
                x.signal()은 정확하게 하나의 suspend된 프로세스를 resume함. suspend된 프로세스가 없으면 아무일도 일어나지 않음