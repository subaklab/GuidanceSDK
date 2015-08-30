# FAQ

## Guidance SDK에서 제공하는 정보는 어떤게 있나요?
Guidance SDK는 다음을 포함하고 있다 :

-	**demo**: Guidance SDK를 이용한 비주얼 트래킹 프로젝트
-	**doc**: API 상세 정보
-	**examples**: USB, UART, ROS에 대한 예제
-	**include**: Guidance SDK의 헤더 파일
-	**lib**: 윈도우용 라이브러리
-	**so**: 리눅스용 라이브러리

Guidance API는 사용자가 `IMU 데이터`, `속도`, `장애물 거리`, `그레이 스케일 이미지`, `depth 이미지`, `울트라소닉 데이터`에 대한 정보를 `USB`와 `UART` 인터페이스로 얻을 수 있다.


## Guidance SDK를 어디에 적용할 수 있나요?
Guidance Core에서 Guidance SDK를 사용하면 다른 하드웨어 플랫폼과 분리가 가능하다. Guidance SDK는 각 운영체제에 대해서 라이브러리 파일을 제공한다. 32비트나 64비트 리눅스와 윈도우에서 어플리케이션을 개발할 수 있다.


## Guidance Core에서 데이터를 얻기 위해서 SDK를 어떻게 사용해야 하나요?
먼저 제공하는 `Developer Guide`를 참고해서 개발환경을 설정한다. 여기에는 드라이버 설치와 프로젝트 속성을 설정하는 방법이 포함되어 있다. 다음으로 Guidance Core를 PC에 연결하고 전원을 넣으면, 어떻게 동작하는지 데모와 예제를 실행할 수 있다.


## 왜 "Device Manager"에서 Guidance device를 볼수 없나요?
먼저 Guidance Core에 전원을 넣었는지 확인하자. 만약 USB로 Guidance Core를 PC에 연결하는 경우, `Guidance Assistant` 소프트웨어가 성공적으로 설치되었는지 검사한다.
만약 Guidance Core를 UART를 통해 PC에 연결하는 경우, `USB_to_RS232` 드라이버가 제대로 설치되었는지 확인한다. 


## SDK의 데모와 예제가 동작하지 않는 경우는요?
USB나 UART 드라이버가 제대로 설치되었는지 확인한다. USB로 PC에 연결한 경우에는 `Guidance Assistant` 소프트웨어를 이용하고 UART인 경우 `Device Manager`에서 검사할 수 있다.

다음으로 Guidance Assistant 소프트웨어에 `DIY` 모드가 설정하고 이미 활성화 되어 있다면 `USB`나 `UART` 인터페이스를 활성화 시킨다.

OpenCV에서 error가 발생한다면, 대부분의 경우 환경변수인 `OPENCVROOT`를 제대로 추가하지 않은 경우다.


## UART를 이용해서 이미지 데이터를 얻을 수 있나요?
할 수 없다. UART의 대역폭의 제한으로 UART를 통해서 이미지 데이터를 전송할 수 없다.


## 모든 그레이 스케일 이미지와 depth 이미지를 동시에 선택할 수 있나요?
할 수 없다. USB의 대역폭의 제한으로 동시에 모든 이미지 데이터를 선택할 수 없다.


## 만약 depth 이미지만 원하는 경우 그레이 스케일 이미지만 선택해야 하나요?
만약 depth 이미지만 원한다면, 그레이 스케일 이미지를 선택할 필요가 없다. 비록 그레이 스케일 임지ㅣ와 depth 이미지는 동일한 구조지만 depth 이미지만 선택하는 것은 가능하다. 기억해야할 사항은 그레이 스케일 이미지에 대한 포인터는 이 경우에 유효하지 않다.


## USB와 UART 데이터를 동시에 얻을 수 있나요?
가능하다. 동시에 양쪽에서 데이터를 얻어올 수 있다. 하지만 USB가 데이터의 모든 타입을 제공하므로 추천하지 않는다.


## 문제가 있을 때 어디서 도움을 받을 수 있나요?
먼저 SDK 패키지에서 제공하는 **`Developer Guide`**를 참조한다. 처음 Guidance SDK를 사용하는데 많은 도움을 얻을 수 있다.

만약 계속 문제가 있다면, 우리 포런에 질문을 올리면 된다. [http://forum.dji.com/](http://forum.dji.com/forum.php?lang=en)


## Guidance SDK를 사용해서 프로젝트를 만드는 방법이 있나요?
Guidance SDK에서 제공한 데모와 예제 프로젝트를 수정해서 여러분의 프로젝트를 만들 수 있다. 아니면 튜토리얼을 하나씩 따라하면서도 가능하다. **`Developer Guide`**에 있는 `How to build a visual tracking project`를 참고하자.


## USB를 사용하는 경우 데이터 전송의 대역폭 제한은 어떻게 되나요?
이미지 데이터의 상위 대역폭 제한은 20hz이며 더 많은 이미지 데이터를 선택하는 경우 떨어질 수 있다. IMU 데이터, 속도 등등의 데이터는 20Hz로 고정되어 있다.


## UART를 사용하는 경우 지원하는 baud rate는?
현재 UART를 사용하는 경우  **115200** baud rate만 지원한다.


## Guidance SDK를 사용하는 카메라 고유 인자를 얻을 수 있나요?
현재 이 기능은 제공하지 않고 있다. Guidance SDK의 다음 버전은 카메라 고유 인자를 얻어오는 인터페이스를 제공할 예정이다.