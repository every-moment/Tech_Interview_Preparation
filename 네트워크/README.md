# 🐼 네트워크


## Blocking vs Non-Blocking, Sync vs Async 에 대해서 각각 연관성 있게 설명하시오.

많이 들어본 용어들이지만 상당히 헷갈리는 개념들이다. 얼핏 보기에 Sync와 Blocking이 비슷하고, Async와 Non-Blocking이 비슷한 것 같다.   
</br>

Blocking과 Non-Blocking, Sync와 Async에 대해서 공부하다보면 항상 보게되는 그림이 있다.    
<div><img src="https://user-images.githubusercontent.com/56947879/135707152-44474515-27a6-4f54-95a2-e7f223e38555.png" align="center" width="40%"></div>
</br>
이 도표에서 Sync & Blocking과 Async & Non-Blocking은 비슷한 것끼리 묶여 있기 때문에 어느 정도 알 것 같기도 하다. 그런데 Sync & Non-Blocking와 Async & Blocking는 과연 어떻게 이해해야 할까?    
Sync & Non-Blocking과 Async & Blocking을 이해하기 위해서는 Sync, Async 그룹과 Blocking, Non-Blocking 그룹의 관심사에 초점을 맞춰볼 필요가 있다.    
</br>
</br>

### Blocking vs Non-Blocking    
> - Blocking과 Non-Blocking의 관심사는 '제어권의 반환'이다.    
> - 호출된 함수가 자신의 작업이 완료될 때까지 호출한 함수에 제어권을 반환하지 않아서 호출한 함수가 그동안 다른 작업을 처리할 수 없다면 Blocking이다.    
> - 반면에, 호출된 함수가 호출한 함수에 즉시 제어권을 반환하여 호출한 함수가 다른 작업을 할 수 있게 한다면 Non-Blocking이다.        
</br>

### Sync vs Async     
> - Sync와 Async의 관심사는 호출된 함수의 '작업 완료를 기다리는지'에 대한 여부이다.    
> - 호출한 함수가 호출된 함수의 작업이 완료되는 것을 기다리면 Sync이다.    
> - 호출한 함수가 호출된 함수의 작업이 완료되는 것을 기다리지 않으면 Async이다. 호출하는 함수는 호출되는 함수에게 callback을 전달하고, 호출되는 함수는 자신의 작업이 완료되면 callback을 실행시킴을 통해 호출하는 쪽에서 호출되는 쪽의 작업 완료 여부에 대해 신경 쓰지 않는다.    
</br>
</br>

### Sync & Non-Blocking    
이제 Sync & Non-Blocking에 대해서 알아보자. 위에서 살펴본 바와 같이 Sync는 호출한 함수가 호출된 함수의 작업 완료를 기다린다. 또한, Non-Blocking은 호출된 함수가 호출한 함수에 즉시 제어권을 반환하여 호출한 함수가 다른 작업을 할 수 있게 한다. 이를 조합해보면 Sync & Non-Blocking는 호출한 함수가 호출된 함수의 작업 완료를 기다리지만 동시에 즉시 제어권을 반환받는다. 따라서 호출한 함수는 다른 작업을 할 수 있지만, 호출된 함수가 작업이 완료됐는지 계속해서 물어보면서 작업 완료를 기다린다.   
</br> 

이에 대해 좀 더 살펴보기 위해 리눅스의 Synchronous Non-Blocking I/O를 보자.    
<div><img src="https://user-images.githubusercontent.com/56947879/135711959-7ebcf563-4f90-472b-92ca-cd5bdf0b2831.png" align="center" width="50%"></div>
Application이 System call을 통해 Kernel에게 제어권을 넘긴다. Kernel은 작업이 완료되지 않았지만 즉시 Application에 제어권을 반환하고, 작업이 완료되지 않았다는 오류(EAGAIN, EWOULDBLOCK)를 반환하여 다시 호출해야 함을 알린다. Application은 계속해서 read, accept같은 함수를 호출하여 I/O를 완료할 수 있는 상태가 되었나 계속해서 물어보면서 작업 완료를 기다린다.    
이를 통해 살펴보면 Kernel이 Application에 즉시 제어권을 반환기 때문에 Non-Blocking이다. 또한, Application이 I/O를 완료할 수 있는 상태가 되었나 계속해서 물어보면서 작업 완료를 기다리기 때문에 Sync이다. 이러한 방식은 Context-Switch로 인해 Cost가 계속해서 발생하긴 하지만, Application은 제어권을 갖고 있기 때문에 작업 완료에 대해 물어보는 중간 중간에 자신의 작업을 처리할 수 있다.    
</br>
</br>

### Async & Blocking    
Async & Blocking에 대해 알아보자. 위에서 살펴본 바와 같이 Async는 호출한 함수가 호출된 함수의 작업 완료를 기다리지 않는다. 또한, Blocking은 호출된 함수가 작업을 완료할 때 까지 호출한 함수에 제어권을 넘기지 않는다. 어차피 Blocking이 되어서 다른 작업을 처리할 수 없는데 작업 완료도 기다리지 않는다는 점에서 이 방식은 이점이 없어보인다.    
이와 같은 경우의 예는 대표적으로 Node.js와 MySQL의 조합이 있다. Async & Non-Blocking을 의도했지만 실제로는 Async & Blocking이 되어버린 케이스다. Node.js에서는 MySQL에 Async로 접근을 하여도 DB 작업을 호출할 때는 MySQL에서 제공하는 드라이버를 호출하게 되는데 이 드라이버가 Blocking방식이다.    
따라서 Async & Blocking는 특출난 이점이 없어서 의도하여 사용할 필요는 없지만, Async & Non-Blocking 방식을 사용하는 과정에서 하나라도 Blocking이 포함되어 있으면 의도와는 다르게 Async & Blocking이 되어버린다고 생각할 수 있다.
</br>
</br>

### 마무리
> - Blocking과 Non-Blocking의 관심사는 '제어권의 반환'. 제어권을 즉시 반환하면 Non-Blocking, 그렇지 않으면 Blocking    
> - Sync와 Async의 관심사는 호출된 함수의 '작업 완료를 기다리는지'에 대한 여부. 작업 완료를 기다리면 Sync, 그렇지 않으면 Async    
</br>
</br>
</br>

## TCP, UDP에 대해서 설명하시오. 그리고 차이점을 설명하시오.

TCP와 UDP는 모두 전송 계층에서 사용되는 프로토콜이다. 전송 계층에서는 데이터를 포트번호에 해당하는 프로세스에 전달하는 것을 담당한다.    
</br>     

#### TCP는     
> - 3 way-handshake를 통해 Connection을 연결하여 양방향 통신을 한다.     
> - 데이터의 순차 전송을 보장한다.     
> - 흐름제어(Flow Control)를 한다. 흐름제어는 데이터 처리 속도를 조절하여 수신자의 버퍼 오버플로우를 방지하는 것이다.     
> - 혼잡제어(Congestion control)를 한다. 네트워크 내에 패킷의 수가 과도하게 증가하는 현상을 혼잡이라고 하고 네트워크의 혼잡을 피하기 위해 송신측에서 보내는 데이터의 전송속도를 강제로 줄이게 작업을 혼잡제어라고 한다.        
> - TCP는 전송의 신뢰성은 보장하지만 3 way-handshake을 통해 매번 Connection을 연결하기 때문에 시간 손실이 발생하고, 패킷을 조금만 손실해도 재전송을 해야한다는 단점이 있다.                
</br>     
   
#### UDP는     
> - TCP보다 신뢰성은 떨어지지만, 전송 속도가 일반적으로 빠르다. TCP처럼 Connection을 연결하지 않고 Connectionless 상태로 통신을 한다.     
> - TCP의 특징인 순차 전송, 흐름제어, 혼잡제어를 보장하지 않는다.     
> - 비교적 데이터의 신뢰성이 중요하지 않을 때 사용된다. 주로 실시간 방송과 온라인 게임에서 UDP가 사용된다.     
</br>

## REST API

REST API란 REST를 기반으로 만들어진 API         
API는 어떠한 응용프로그램에서 데이터를 주고 받기 위한 방법
</br>

REST란
 - HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)을 명시하고, HTTP Method(POST, GET, PUT, DELETE)를 통해 해당 자원에 대한 CRUD Operation을 적용하는 것을 의미한다.    
 - REST는 기본적으로 웹의 기존 기술과 HTTP 프로토콜을 그대로 활용하기 때문에 웹의 장점을 최대한 활용할 수 있는 아키텍처 스타일이다
 - 즉, REST는 자원 기반의 구조(ROA, Resource Oriented Architecture) 설계의 중심에 Resource가 있고 HTTP Method를 통해 Resource를 처리하도록 설계된 아키텍쳐를 의미한다.
</br> 

REST API 중심 규칙
 - 1. URI는 정보의 자원을 표현해야 한다. (리소스명은 동사보다는 명사를 사용)
 - 2. 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE 등)로 표현
</br>

URI 설계 시 주의할 점
 - 1. 슬래시 구분자(/)는 계층 관계를 나타내는 데 사용
 - 2. URI 마지막 문자로 슬래시(/)를 포함하지 않는다.
 - 3. 하이픈(-)은 URI 가독성을 높이는데 사용 & 언더바(_)는 URI에 사용하지 않는다.
 - 4. URI 경로에는 소문자가 적합하다. 대문자 사용은 피해야 한다.
 - 5. 파일 확장자는 URI에 포함시키지 않는다.  Accept header를 사용하도록 하자.
</br>

URI vs URL
 - URI: Uniform Resource Identifier의 약자로 인터넷상 포현할수 있는 모든 리소스를 의미
 - URL: Uniform Resource Locator의 약자로 URI에서 네트워크상 경로, 어떤 자원의 위치에 대한 절대경로값
</br>
</br>
</br>

## Request Header, Response Header 에 대해서 아는 대로 쓰시오. (요청헤더, 응답헤더) 

[질문에 대한 정리 : Http - Response Request](https://velog.io/@minyul/Http-Response-Request)

</br>

## DNS 에 대해서 설명하시오.
