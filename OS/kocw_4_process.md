# 프로세스 관리

## 프로세스의 개념

<img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/232839873-398c7323-1da8-4764-8314-c0f8731722f4.png">

- 프로세스(Process)
    - 실행 중인 프로그램
- 프로세스의 문맥(context)
    - 프로세스가 특정 시점에서 어떤 상태에서 수행되고 있는지에 대한 정보
    1. CPU 수행 상태를 나타내는 하드웨어 문맥
        - Program Counter, 여러 register
            - PCB의 PC는 메모리의 data 영역에 있고, CPU의 PC는 실제 레지스터 PC
    2. 프로세스의 주소 공간
        - code, data, stack 영역
    3. 프로세스 관련 커널 자료 구조
        - PCB, Kernel stack

## 프로세스의 상태

- 프로세스는 상태를 크게 3-4가지로 구분
    
    <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/232840148-5e29e295-edc8-4f45-9d55-4c907c6aba81.png">
    
    1. Running - 실행
        - CPU를 잡고 instruction을 수행중인 상태
    2. Ready - 준비
        - CPU를 기다리는 상태(모든 준비가 다 됨)
    3. Blocked(wait, sleep) - 봉쇄
        - CPU를 주어도 당장 instruction을 수행할 수 없는 상태
        - 프로세스 자신이 요청한 event가 즉시 만족되지 않아 기다리는 상태
        - 자신이 요청한 event가 만족되면 다시 Ready 상태로 넘어감
    4. New
        - 프로세스가 생성중인 상태. 아직 메모리 획득 못함
    5. Terminated
        - 수행이 끝난 상태. 아직 프로세스와 관련된 자료구조 완전히 정리 못함
    6. Suspended(stopped)
        - 외부적인 이유로 프로세스의 수행이 정지된 상태
        - 프로세스가 통째로 디스크에서 swap out됨
        - 외부에서 resume해 주어야 Active될 수 있음
        - 중기 스케줄러로 이 상태가 추가됨
    
    → Ready, Blocked은 기다리므로 Queue에서 관리됨
    
    → 이 Queue는 OS의 Kernel이 data영역에 자료구조로 Queue를 만들어 놓고 관리
    

## 프로세스 제어블록(Process Control Block: PCB)

- 운영체제가 각 프로세스를 관리하기 위해 프로세스당 유지하는 정보(문맥)
- 4가지 구성요소를 구조체로 유지하며 가짐
    
    <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/232840313-3c54b49b-d47d-47cb-91b8-fd83b6debbd0.png">
    
    1. OS가 관리상 사용하는 정보
    2. CPU 수행 관련 하드웨어 값
    3. 메모리 관련
    4. 파일 관련

## 문맥교환(context switch)

- CPU를 한 프로세스(사용자 프로그램)에서 다른 프로세스(사용자 프로그램)로 넘겨주는 과정. CPU 디스패치
- 이때 OS는 다음을 실행
    1. CPU를 내어주는 프로세스의 상태를 그 프로세스의 PCB에 저장
    2. CPU를 새롭게 얻는 프로세스의 상태를 PCB에서 읽어옴
- System call이나 Interrupt 발생 시, 반드시 context switch가 일어나는 것은 아님
    
    <img width="350" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/232840496-bbc44000-99b0-45e8-b7f9-64e2a1ba3c06.png">
    
    - 1의 경우(모드 교환)도 PCB를 저장해야하지만 문맥교환하는 2의 경우(문맥 교환)가 오버헤드가 훨씬 큼
        
        → 문맥교환이 일어나면 A가 사용하던 cache memory를 다 지워야함
        

## 프로세스를 스케줄링하기 위한 큐

- 프로세스들은 각 큐(Device Queue, Ready Queue)들을 오가며 수행
    
    <img width="400" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/232840778-486ad502-f8f9-4d46-8b55-27a6ba17f044.png">
    
    1. Job Queue
        - 현재 시스템 내에 있는 모든 프로세스의 집합(Ready+Device Queue+…)
    2. Ready Queue
        - 현재 메모리 내에 있으면서 CPU를 잡아서 실행되기를 기다리는 프로세스의 집합
    3. Device Queues
        - IO device의 처리를 기다리는 프로세스의 집합
    4. 소프트웨어 자원을 위한 Queue
        - ex. 공유 데이터는 일관성을 위해 한 시점에 한 프로세스만 접근 가능

## 스케줄러(Scheduler)

- 어떤 프로세스에게 자원을 할당하지 결정하는 OS 커널의 코드
1. Long-term schedular(장기 스케줄러 or job scheduler)
    - 시작 프로세스 중 어떤 것들을 **ready queue**로 보낼 지 결정
    - 프로세스에 <b>memory(및 각종 자원)</b>을 주는 문제
    - degree of Multiprogramming(메모리에 여러 프로그램이 올라가는 것)을 제어
    - time sharing system에는 장기 스케줄러가 없고 무조건 ready 상태
2. Short-term schedular(단기 스케줄러 or CPU scheduler)
    - 어떤 프로세스를 다음번에 **running**시킬지 결정
    - 프로세스에 **CPU**를 주는 문제로 자주 일어남 → 빨라야 함
3. Medium-term schedular(중기 스케줄러 or Swapper)
    - 다 메모리에 올려놓은 후, 여유 공간 마련을 위해 프로세스를 통째로 메모리에서 디스크로 쫓아냄
    - 프로세스에게서 **memory**를 뺏는 문제(swap out)
    - time sharing system에서 사용
    - 봉쇄 상태 프로세스 → 타이머 인터럽트를 통해 준비 큐로 이동한 프로세스 순으로 swap out

## 프로세스의 생성

- (부모)프로세스가 (자식)프로세스를 복제 생성
    - 자식을 먼저 종료해야 부모가 종료 가능
    - 자식은 부모의 주소 공간을 그대로 복사한 후, 새로운 프로그램의 주소 공간으로 덮어씌워 실행
- 자원 획득 방법
    1. OS에게 직접 자원 할당받음
    2. 부모 프로세스와 자원 공유
- 프로세스가 수행되는 모델
    1. 부모와 자식이 공존하며 수행하는 모델
        - 자식과 부모가 CPU를 획득하기 위해 경쟁
    2. 자식이 종료될 때까지 기다리는 모델
        - 자식이 종료될 때까지 부모는 봉쇄 상태(부모와 자식 프로세스 간의 동기화 가능)
- 프로세스의 종료
    1. 마지막 명령을 수행한 후 자발적 종료
    2. 부모가 자식을 강제로 종료하는 비자발적 종료

## 프로세스 간의 협력

- 원칙적으로 프로세스 간의 주소 공간 참조 불가능
- IPC
    - 하나의 컴퓨터 안에서 실행 중인 서로 다른 프로세스 간에 발생하는 통신
    - 동기화를 위해 한 데이터는 한 프로세스만 접근 가능
    1. 메시지 전달 방식
        - 프로세스 간에 공유 데이터 사용하지 않음. 커널을 통한 메시지로 통신
    2. 공유 메모리 방식
        - 프로세스들이 주소 공간의 일부를 공유. 독립적인 주소 공간이므로 시스템 콜 사용

## Thread(lightweight process)

- 프로세스 내부의 CPU 수행 단위
- 프로세스 하나의 PCB에 스레드를 여러개 만들어 각각의 PC, register를 사용(PCB는 한개)
    
    → 프로세스 주소 공간을 하나만 만들어 같은 프로그램을 여러개 실행할 수 있음
    
- 공유O: 프로세스의 메모리 주소 공간(code, data 영역), 프로세스 상태, 프로세스 자원, OS 자원
- 공유X: CPU수행과 관련된 정보(PC, register, Stack)는 별도로 사용

<br/>

- Thread의 구성
    
    → program counter, register set, stack space
    
- 장점
    1. 응답성
        
        다중 스레드 구조에서는 하나의 서버 스레드가 blocked 상태인 동안에도, 동일한 테스크 내의 다른 스레드가 실행되어 빠른 처리 가능
        
    2. 자원 공유
        
        code, data 등의 자원을 공유해서 효율적으로 사용 가능
        
    3. 경제성
        
        프로세스 생성과 CPU switching은 오버헤드가 큼 → 이것을 방지할 수 있음
        
        동일한 일을 수행하는 다중 스레드가 협력하여 높은 처리율과 성능 향상을 얻을 수 있음
        
    4. 멀티 프로세스에서 병렬성 높임
        
        멀티 프로세스 환경에서 스레드를 사용하면 각각의 프로세스에서 여러 스레드를 동시에 사용할 수 있으므로 병렬성을 높을 수 있음
        
- 구현 방법
    1. Kernel Threads
        
        스레드가 여러개 있다는 사실을 Kernel이 알고있음
        
        → 스레드를 넘어가는 것을 Kernel이 CPU 스케줄링하는 것처럼 넘김
        
    2. User Threads
        
        라이브러리를 통해 지원. Kernel은 Thread가 여러 개인 것을 모름
        
    3. Real Time Threads
 
</br>

### 레퍼런스

---

- 반효경, 운영체제, KOCW, [http://www.kocw.net/home/search/kemView.do?kemId=1046323](http://www.kocw.net/home/search/kemView.do?kemId=1046323)
- 반효경, 운영체제와 정보기술의 원리
