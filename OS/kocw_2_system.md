# 컴퓨터 시스템의 동작 원리

## 컴퓨터 시스템의 구조

<img width="500" alt="os_2_1" src="https://user-images.githubusercontent.com/55101567/232325471-2956eb32-268b-4e86-b655-fd689c49e538.png">


1. CPU
    - 연산 수행
    - 메모리의 instruction만 실행 = 메모리만 직접 접근
    - OS는 interrupt가 들어와야지만 CPU 점유 가능
    1. Interrupt line
        - Interrupt를 받는 곳
    2. mode bit
        - CPU에 실행되는 것이 OS인지 사용자 프로그램인지 구분해 접근 권한 제어
        - 1 : 사용자 모드(사용자 프로그램 수행)
        - 0 : 커널(모니터) 모드(OS 코드 수행)
            - 특권 명령 : 커널 모드에서만 수행 가능한 명령어. 시스템의 보안과 관련된 명령어
                
                ex) 입출력 명령, 기준/한계 레지스터 세팅
                
    3. registers
        - 데이터 저장하는 작은 공간으로 메모리보다 빠름
2. DMA
    - CPU이외에 메모리 접근이 가능한 장치
        - 모든 메모리 접근 연산을 CPU가 처리하면 CPU 효율성이 떨어짐
            
            → DMA가 IO장치들에게 CPU가 자주 interrupt되는 것을 방지
            
    - byte가 아니라 block 단위 연산 후, interrupt를 발생시켜 작업 완료 알림
3. Memory
    - 프로그램을 실행하기 위해 반드시 메모리에 올려야함
    - CPU만 접근 가능
4. Timer
    - 정해진 시간이 흐른 뒤 OS에게 제어권이 넘어가도록 interrupt 발생
    - CPU를 특정 프로그램이 독점하는 것으로부터 보호
5. Controller
    - 각 하드웨어 장치마다 존재하면서 이들을 제어하는 작은 CPU
    - 제어 정보를 위해 register 및 local buffer를 가짐
    - Device controller는 IO작업이 끝나면, interrupt로 CPU에게 알림
6. Kernel
    - OS는 항상 자원 관리를 해야하므로 메모리에 상주해야함
    - OS의 모든 코드를 다 메모리에 상주할 수 없으므로 핵심적인 부분만 항상 메모리에 있음 ⇒ 커널

## CPU 연산과 IO연산

- A 프로그램이 CPU를 사용하고 B 프로그램이 하드디스크에서 읽기를 하면, 동시에 작업 가능
    1. B는 시스템 콜을 통해 CPU에 interrupt 발생
    2. B의 상태를 PCB에 저장하고 CPU는 OS에 넘어가 인터럽트 처리 루틴으로 이동
    3. B의 일은 Device Controller에게 넘겨지고 장치에서 로컬버퍼로 데이터를 읽어옴
    4. 데이터를 읽어오는 동안 CPU는 A 프로그램에게 넘겨져  실행됨
    5. 원하는 데이터를 로컬 버퍼로 다 읽어오면, Device Controller가 CPU에 interrupt를 발생시킴
    6. CPU는 interrupt line에 신호가 들어오면, 하던 일을 멈추고 CPU의 점유를 OS에게 줌
        1. 이때 A의 실행 상태를 A의 PCB에 저장후 OS에게 CPU 넘겨줌
    7. OS는 interrupt의 일을 처리(B에 데이터 전달) 후 A에게 다시 CPU 넘겨줌
        1. A의 PCB로부터 CPU상에 복원해 이어서 실행됨
- 사용자 프로그램이 IO 연산 요청
    1. 사용자 프로그램이 trap을 사용해 OS로 CPU 제어권 넘어감
    2. 인터럽트 벡트의 특정 위치로 이동하여 인터럽트 서비스 루틴으로 이동
    3. 올바른 요청인지 확인 후 IO 수행
    4. IO 완료 시 controller가 CPU에게 interrupt를 발생해 완료 알림
    5. 사용자 프로그램으로 CPU 넘어감

## Interrupt

- 인터럽트 처리루틴
    - OS 커널 내에 존재해 각 interrupt에 대한 업무를 정의
    - 실제 처리해야할 코드 = 커널 함수
- 인터럽트 벡터
    - 인터럽트 종류마다 번호를 정해, 번호에 따라 처리해야 할 코드가 위치한 부분을 기리키고 있는 자료구조
- 인터럽트 핸들링
    - 인터럽트가 발생한 경우에 처리해야 할 일의 절차
    - 프로세스 제어블록(PCB)  
        <img width="250" alt="os_2_2" src="https://user-images.githubusercontent.com/55101567/232325469-16f9801a-b38e-478f-a623-afa84e6a5a3e.png">
        
        - 현재 시스템 내에서 실행되는 프로그램들을 관리하는 자료구조
        - 각 프로그램마다 하나씩 존재, 어떤 부분을 실행 중이었는지 저장
1. Interrupt(하드웨어 인터럽트)
    - Controller 등 하드웨어 장치가 CPU의 interrupt line을 세팅
2. trap(소프트웨어 인터럽트)
    - 소프트웨어가 CPU의 interrupt line을 세팅
    1. 예외상황
        - division by zero 같은 비정상적인 작업을 시도하거나, 권한이 없는 작업을 시도할 때 발생
    2. 시스템 콜
        - 사용자 프로그램이 OS 내부에 정의된 코드(커널 함수)를 실행하고 싶을 때, OS에 서비스 요청

## IO 구조

1. 동기식(synchronous) 입출력
    - 입출력 발생 시, 입출력 작업이 완료된 후에야 다음 작업을 수행할 수 있는 방식
        
        → 자원 낭비
        
    - 그래서 일반적으로 A 프로그램이 IO 수행 시, CPU를 다른 B 프로그램에게 이양해 CPU가 계속 일할 수 있도록 관리. 이때 A 프로그램은 봉쇄 상태(blocked state)로 전환됨.
    - 이때 여러 입출력이 동시에 요청되거나 처리될 수 있어 동기화 문제가 일어날 수 있음
        
        → 장치별로 Queue를 두어 순서대로 처리하므로서 동기성 보장
        
2. 비동기식(asynchronous) 입출력
    - 입출력 발생 시, 입출력 작업이 끝나길 기다리지 않고 CPU의 제어권을 입출력을 호출한 그 프로그램에게 바로 부여하는 방식
    - 읽는 데이터와 상관 없는 연산이나 쓰기 요청에서 사용됨

## 저장장치의 구조

1. 주기억장치
2. 보조기억장치
    1. 파일 시스템용
    2. 스왑 영역용
        - 메모리의 용량이 작기때문에, 당장 필요한 부분만 메모리에 올리고 그렇지 않은 부분은 디스크의 스왑영역에 내려놓음 = 스왑 아웃
        - 해당 프로그램이 종료될 때 삭제하는 메모리의 연장 공간
- 계층 구조
    
    <img width="350" alt="os_2_3" src="https://user-images.githubusercontent.com/55101567/232325465-d55a4733-8f28-4168-9dd4-cd845ca23e4e.png">

    
    위에 올라갈수록, 빠르고, 비싸고, 적은 용량
    
    - 캐싱 기법
        - 자주 사용되거나 당장 사용될 정보를 저장해 속도 완충

### 용어

---

**device driver(장치 구동기)**

- OS 코드 중 각 장치별 처리 루틴 → software

**device controller(장치 제어기)**

- 각 장치를 통제하는 일종의 작은 CPU → hardware

### 레퍼런스

---

- 반효경, 운영체제, KOCW, [http://www.kocw.net/home/search/kemView.do?kemId=1046323](http://www.kocw.net/home/search/kemView.do?kemId=1046323)
- 반효경, 운영체제와 정보기술의 원리