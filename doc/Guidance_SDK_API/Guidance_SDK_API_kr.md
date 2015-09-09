# 1 배경

이 가이드에서는 여러분이 다음을 가지고 있다고 가정한다.

- Guidance 시스템,

- OpenCV가 설치된 컴퓨터,

그리고 여러분은:

- 리눅스 프로그래밍에 익숙하고,

- 혹은 윈도우 프로그래밍과 Microsoft Visual Studio에 익숙하다.


# 2 소개

이 섹션에서는 Guidance SDK의 구조를 소개한다. SDK는 3개 계층으로 나눠져 있다:

![](./image/Guidance_SDK_API3987.png)

- **Application:** 이 계층은 HAL 계층으로부터 데이터를 처리한다. 개발자가 작성한다.
- **HAL:** 이 계층은 Driver 계층과 통신하여 데이터를 패킹하거나 파싱한다. 예제 코드(UART)나 SDK 라이브러리(USB)로 구현된다. e.g. _libDJI\_guidance.so_.
- **Driver:** 이 계층은 Guidance 시스템에서 USB/UART로 데이터를 받는다.OS나 써드파티가 구현한다. e.g. _libusb_.

## 2.1 인터페이스
Guidance SDK는 2개 통신 프로토콜을 제공한다 : USB와 UART

### 2.1.1 USB

제공하는 데이터 타입은 속도 데이터와 장애물 거리 데이터, IMU 데이터, 울트라소닉 데이터, 그레이 스케일 이미지, Depth 이미지이다.

USB를 통해 데이터를 받기 위해 2가지 방법이 있다.

1. Guidance Assistant 소프트웨어

	Guidance Assistant 소프트웨어를 이용해서 "DIY->API->USB" 탭에 있는 데이터를 받을 수 있다.
	
	- Guidance를 USB 케이블로 PC와 연결하고 전원을 켠다.
	- "Enable" 체크박스 선택
	- 원하는 데이터 선택

	**주의:** 가용한 대역폭은 이미지 데이터의 선택과 출력 대역폭이 반대다. 받은 이미지 데이터의 선택과 출력 대역폭은 저장되어 Guidance 시스템을 켰다가 켜도 계속 유지된다.

	![](./image/Guidance_SDK_API5146.png)

2. Guidance API

	Guidance API를 이용해서 데이터를 받을 수 있다. 이러한 API 기능을 "select"라는 이름을 붙여서 구별한다.

	**주의:** 만약 사용자가 Guidance API 함수를 사용해서 이미지 데이터와 출력 빈도를 받는다면, Guidance 시스템이 켜져있는 동안 Guidance Assistant 소프트웨어에서 만들어진 데이터 섹션을 임시로 덮어쓰기할 것이다. 하지만 "USB"탭에서 "Enable"옵션을 해제하지 않는다면, Guidance API를 통해 만들어진 데이터 섹션은 Guidance 시스템에 저장된 데이터 수신 옵션을 영구히 변경하지 않는다.

### 2.1.2 UART

지원하는 데이터 타입은 속도 데이터, 장애물 거리 데이터, IMU 데이터, 울트라소닉 데이터이다.

1. Subscribe Data

	UART 데이터를 받기위해 Guidance assistant 소프트웨어를 사용할 수도 있다.  "DIY->API->UART" 페이지에 이 섹션을 유효하게 만든다. "UART"탭에서 Enable" 옵션을 해제하지 않으면, USB와 같이 설정은 Guidance Core에 저장된다.

	![](./image/Guidance_SDK_API6086.png)

2. Protocol 상세

Protocol 프레임 포맷:

| SOF | LEN | VER | RES | SEQ | CRC16 | DATA | CRC32 |
| --- | --- | --- | --- | --- | --- | --- | --- |

Protocol 프레임 설명:

| Field | Byte Index | Size（bit） | Description |
| --- | --- | --- | --- |
| SOF | 0 | 8 | Frame start number, fixed to be 0xAA |
| LEN | 1 | 10 | Frame length, maximum length is 1023 bytes |
| VER | 1 | 6 | Version of the protocol |
| RES | 5 | 40 | Reserved bits, fixed to be 0 |
| SEQ | 8 | 16 | Frame sequence number |
| CRC16 | 10 | 16 | Frame header CRC16 checksum |
| DATA | 12 | --① | Frame data, maximum length  1007bytes |
| CRC32 | --② | 32 | Frame CRC32checksum |

1. 프레임 데이터 크기는 가변적이며, 1007이 최대 크기다.
2. 이 필드의 인덱스는 데이터 필드의 길이에 의존한다.

데이터 필드 포맷:

| COMMAND SET | COMMAND ID | COMMAND DATA |
| --- | --- | --- |

데이터 필드 설명:

| Data Field | Byte Index | Size（byte） | Description |
| --- | --- | --- | --- |
| COMMAND SET | 0 | 1 | Always 0x00 |
| COMMAND ID | 1 | 1 | e\_image: 0x00e\_imu: 0x01e\_ultrasonic: 0x02e\_velocity: 0x03e\_obstacle\_distance: 0x04 |
| COMMAND DATA | 2 | -- | Data body |

## 2.2 데이터 타입

지원하는 데이터 타입의 각각은 아래와 같다.

- **Velocity Data:** 속력 정보를 출력한다. 단위는 "millimeter/second"이며 빈도는 20Hz이다.

- **Obstacle Distance Data:** 5개 방향에 대해서 장애물 거리를 출력한다. 단위는 **centimeter**이며 빈도는 20Hz이다.

- **IMU Data:** IMU 데이터를 출력하며 가속도(meter/second)와 attitude(quaternion포맷) 데이터를 포함한다. 빈도는 20Hz이다.

- **Ultrasonic Data:** 5개 방향에 대한 울트라소닉 데이터를 출력한다. 여기에는 장애물의 거리(미터)와 데이터의 신뢰성을 포함하고 있다. 빈도는 20Hz이다.
Outputs ultrasonic data for five directions, including obstacle distance (in unit of meter) and reliability of the data. The frequency is 20 Hz.

- **Greyscale Image:** 5개 방향에 대해서 그레이 스케일 이미지를 출력한다. 이미지의 크기는 각 센서에 대해 320\*240 바이트이다. 기본 빈도는 20Hz이고 Guidance API를 이용해서 낮출 수 있다.

- **Depth Image:** 5개 방향에 대해서 depth 이미지를 출력한다. 이미지의 크기는 각 방향에 대해서 320\*240\*2 바이트이다. 기본 빈도는 20hz이고 Guidance API를 이용해서 낮출 수 있다.
  
  **주의:** 최고의 성능을 위해서, 처리하는데 시간이 오래 걸릴꺼 같은 큰 사이즈의 데이터의 경우 데이터를 바로 처리하는 대신에 복사하는 것을 권한다.

# 3 시작하기

Guidance SDK는 Guidance 시스템으로부터 데이터를 얻기 위한 예제를 제공한다. 이 섹션은 예제를 실행하는 방법을 가이드한다.

### 3.1 리눅스에서 USB 예제 실행

1. **환경설정**

	Guidance SDK는 Guidance 시스템에서 데이터를 읽기 위해서 _libusb-1.0_ 라이브러리를 사용한다. _libusb-1.0_ 라이브러리를 컴파일하고 설치하기 위해서 _[http://www.libusb.org](http://www.libusb.org/)_를 참조하기 바란다.

2. **관련 파일 복사**

	Makefile은 테스트해서 제공되었다. 사용자는 변경없이 에제 코드를 실행할 수 있다.

	자신의 프로젝트에서 Guidance를 사용하기 위해서는 사용자는 아래와 같은 지시를 따라야 한다:

	- _libDJI\_guidance.so_를 여러분 프로젝트의 "library file path"에 복사한다.

	- _DJI\_guidance.h_를 "header file path"에 복사한다.

	- 여러분 프로젝트의 Makefile에 해당 라이브러리를 아래와 같이 추가한다.
	```
	LDFLAGS = -Wl,-rpath,./ -lpthread -lrt -L./ -L/usr/local/lib/ -l **DJI\_guidance** -lusb-1.0
	```


3. **예제 프로젝트 컴파일**

	![](./image/Guidance_SDK_API9210.png)

	**주의:** 기본 makefile은 설치된 OpenCV가 없다고 가정하고 _Makefile\_noOpenCV_를 사용한다. 각자 자신의 환경에 따라서 작성하는 동안 makefile을 지정할 수 있다. 만약 이미 OpenCV가 설치되어 있다면, 다른 makefile을 사용하자 :

	~~~
	$ make –f Makefile
	~~~


4. **USB를 통해 Guidance 연결해서 실행하기**

	이 예제를 실행하는데 root 권한이 필요하다:

	~~~
	$ sudo ./guidance_example
	~~~

	![](./image/Guidance_SDK_API9567.png)

### 3.2 윈도우에서 USB 예제 실행

1. **환경설정**

	Guidance SDK는 _libusb_ 라이브러리를 사용해서 Guidance 시스템의 데이터를 읽는다. Guidance Assistant 소프트웨어가 제대로 설치되었는지 확인하자. DJI USB 드라이버는 포함되어 있다.

2. **Visual Studio 설정**

	각기 다른 Visual Studio 버전에 대해서 테스트 후 솔루션을 제공한다. 사용자는 변경없이 예제코드를 실행할 수 있다.

	자신의 프로젝트에서 Guidance를 사용하려면,  examples\usb\_example 폴더에 제공하는 속성 시트를 사용하는 방법을 추천한다. 사용자는 헤더 디렉토리와 라이브러리 파일만 변경하면 된다.

	~~~ xml
	<?xml version="1.0" encoding="utf-8"?>
	<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
	  <ImportGroup Label="PropertySheets" />
	  <PropertyGroup Label="UserMacros" />
	  <PropertyGroup />
	  <ItemDefinitionGroup>
	    <ClCompile>
	      <AdditionalIncludeDirectories>$(SolutionDir)..\..\include</AdditionalIncludeDirectories>
	      <AdditionalOptions>/DWIN32 %(AdditionalOptions)</AdditionalOptions>
	    </ClCompile>
	    <Link>      <AdditionalLibraryDirectories>$(SolutionDir)..\..\lib\2010\x86;</AdditionalLibraryDirectories>      <AdditionalDependencies>DJI_guidance.lib;%(AdditionalDependencies)</AdditionalDependencies>
	    </Link>
	  </ItemDefinitionGroup>
	  <ItemGroup />
	</Project>
	~~~

	다른 방안으로, 아래와 같이 파일을 복사해서 Visual Studio를 설정할 수 있다 :

	- _DJI\_guidance.dll_와 _DJI\_guidance.lib_를 프로젝트의 "library file path"로 복사한다.

	- _DJI\_guidance.h_를 "header file path"로 복사한다.

	- _DJI\_guidance.lib_를 Visual Studio 프로젝트의 additional dependencies에 추가한다.

	![](./image/Guidance_SDK_API11350.png)

3. **예제 프로젝트 컴파일**

	Visual Studio를 이용해서 에제 프로젝트를 컴파일한다.
	
4. **테스팅을 위해 Guidance 시스템을 연결**

	![](./image/Guidance_SDK_API11483.png)

### 3.3 UART 예제를 리눅스 환경에서 실행

1. **UART 데이터 받아오기**

	UART 데이터 받아오기는 섹션 2.1.2를 참조하라.

2. **예제 프로젝트를 컴파일**

	![](./image/Guidance_SDK_API11655.png)

3. **테스팅을 위해 Guidance 시스템 연결**

	![](./image/Guidance_SDK_API11699.png)

### 3.4 UART 예제를 윈도우 환경에서 실행

1. **UART 데이터 받아오기**

	UART 데이터 받아오기는 섹션 2.1.2를 참조하라.

2. **예제 프로젝트를 컴파일**

	Visual Studio를 이용해서 예제 프로젝트를 컴파일한다.

3. **테스팅을 위해 Guidance 시스템 연결**

	![](./image/Guidance_SDK_API11973.png)


# 4 자료구조

## e\_sdk\_err\_code

**설명:** SDK의 error 코드 정의

~~~ cpp
enum e_sdk_err_code
{
    e_sdk_no_err = 0,					// error없이 성공
    e_load_libusb_err,					// libusb 라이브러리 로드 error
    e_sdk_not_inited,					// SDK 소트프웨어가 아직 준비되지 않음 
    e_guidance_hardware_not_ready,		// Guidance 하드웨어가 아직 준비되지 않음 
    e_disparity_not_allowed,			// If work type is standard, disparity is not allowed to be selected 
    e_image_frequency_not_allowed,		// Image frequency must be one of the enum type e_image_data_frequecy 
    e_config_not_ready,					// Get config including the work type flag, before you can select data 
    e_online_flag_not_ready,			// Online flag is not ready 
    e_max_sdk_err = 100					// Allow 100 error max.
};
~~~


## e\_vbus\_index

**설명:** vbus의 논리적 방향을 정의. i.e. 선택된 Guidance 센서


~~~ cpp
enum e_vbus_index
{
    e_vbus1 = 1,        // logic direction of vbus
    e_vbus2 = 2,        // logic direction of vbus
    e_vbus3 = 3,        // logic direction of vbus
    e_vbus4 = 4,        // logic direction of vbus
    e_vbus5 = 0         // logic direction of vbus
};
~~~

## e\_image\_data\_frequecy

**설명:** 이미지 데이터의 빈도 정의

~~~ cpp
enum e_image_data_frequecy
{
    e_frequecy_5  = 0,  // frequecy of image data 
    e_frequecy_10 = 1,  // frequecy of image data 
    e_frequecy_20 = 2   // frequecy of image data 
};
~~~

## user\_callback

- **설명:** callback 함수 프로토타입
- **인자:** 데이터 타입을 식별하기 위해서 `event_type`에서 사용: 이미지, imu, 울트라소닉, 속력 혹은 장애물 거리; `data_len` 입력 데이터의 크기; `data` GUIDANCE에서 읽은 입력 데이터
- **Return:** `error code`. error가 발생하면 0이 아닌 값.

~~~ cpp
typedef int (*user_call_back)( int event_type, int data_len, char *data );
~~~


## e\_guidance\_event

**Description:** Define event type of callback.

~~~ cpp
enum e_guidance_event
{
    e_image = 0,            // called back when image comes 
    e_imu,                  // called back when imu comes 
    e_ultrasonic,           // called back when ultrasonic comes 
    e_velocity,             // called back when velocity data comes 
    e_obstacle_distance,    // called back when obstacle data comes 
    e_event_num
};
~~~

## image\_data

**Description:** Define image data structure. The center of depth image coincides with the corresponding left greyscale image's.

~~~ cpp
typedef struct _image_data
{
    unsigned int     frame_index;   // frame index 
    unsigned int     time_stamp;    // time stamp of image captured in ms 
    char      *m_greyscale_image_left[CAMERA_PAIR_NUM];   // greyscale image of left camera 
    char      *m_greyscale_image_right[CAMERA_PAIR_NUM];  // greyscale image of right camera 
    char      *m_depth_image[CAMERA_PAIR_NUM];   // depth image in meters
}image_data;
~~~

## ultrasonic\_data

**Description:** Define ultrasonic data structure. Unit is `mm`.

~~~ cpp
typedef struct _ultrasonic_data
{
    unsigned int     frame_index;    // corresponse frame index 
    unsigned int     time_stamp;     // time stamp of corresponse image captured in ms 
    unsigned short   ultrasonic[CAMERA_PAIR_NUM];    // distance in mm
    unsigned short   reliability[CAMERA_PAIR_NUM];   // reliability of the distance data 
}ultrasonic_data;
~~~

## velocity

**Description:** Define velocity structure. Unit is `mm/s`.

~~~ cpp
typedef struct _velocity
{
    unsigned int     frame_index;        // corresponse frame index 
    unsigned int     time_stamp;         // time stamp of corresponse image captured in ms 
    short            vx;                 // velocity of x in mm/s
    short            vy;                 // velocity of y in mm/s 
    short            vz;                 // velocity of z in mm/s 
}velocity;
~~~


## obstacle\_distance

**Description:** Define obstacle distance structure. Unit is `cm`.

~~~ cpp
typedef struct _obstacle_distance
{
    unsigned int     frame_index;       // corresponse frame index 
    unsigned int     time_stamp;        // time stamp of corresponse image captured in ms 
    unsigned short   distance[CAMERA_PAIR_NUM];     // distance of obstacle in cm
}obstacle_distance;
~~~

## imu

**Description:** Define imu structure. Unit of acceleration is `m/s^2`.

~~~ cpp
typedef struct _imu
{
    unsigned int     frame_index;             // corresponse frame index 
    unsigned int     time_stamp;              // time stamp of corresponse image captured in ms 
    float            acc_x;                   // acceleration of x m/s^2
    float            acc_y;                   // acceleration of y m/s^2
    float            acc_z;                   // acceleration of z m/s^2
    float            q[4];                    // quaternion: [w,x,y,z]
}imu;
~~~


# 5 API

## Overview

The GUIDANCE API provides configuration and control methods for GUIDANCE with C interface.


Here is an overview of the key methods available in this API:

~~~ cpp
// Guidance initialization
SDK_API int reset_config( void );
SDK_API int init_transfer( void );

// Guidance subscribe data
SDK_API void select_imu( void );    
SDK_API void select_ultrasonic( void );
SDK_API void select_velocity( void );   
SDK_API void select_obstacle_distance( void ); 
SDK_API int set_image_frequecy( e_image_data_frequecy frequecy ); 
SDK_API int select_depth_image( e_vbus_index camera_pair_index ); 
SDK_API int select_greyscale_image( e_vbus_index camera_pair_index, bool is_left ); 

// Guidance set event
SDK_API int set_sdk_event_handler( user_call_back handler );

// Guidance transfer control
SDK_API int start_transfer( void );
SDK_API int stop_transfer( void );
SDK_API int release_transfer( void );

// Guidance get status 
SDK_API int get_online_status( int online_status[CAMERA_PAIR_NUM] );
~~~


## Method

### reset\_config

- **Description:** Clear the subscribed configure, if you want to subscribe the different data from last time.
- **Parameters:** NULL
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int reset_config ( void );
~~~


### init\_transfer

- **Description:** Initialize the GUIDANCE and connect it to PC.
- **Parameters:** NULL
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int init_transfer ( void );
~~~

### select\_imu

- **Description:** Subscribe to imu.
- **Parameters:** NULL
- **Return:** NULL

~~~ cpp
SDK_API void select_imu ( void );
~~~

### select\_ultrasonic

- **Description:** Subscribe to ultrasonic.
- **Parameters:** NULL
- **Return:** NULL

~~~ cpp
SDK_API void select_ultrasonic ( void );
~~~

### select\_velocity

- **Description:** Subscribe to velocity data, i.e. velocity of GUIDANCE in body coordinate system.
- **Parameters:** NULL
- **Return:** NULL

~~~ cpp
SDK_API void select_velocity ( void );
~~~


### select\_obstacle\_distance

- **Description:** Subscribe to obstacle distance, i.e. distance from obstacle.
- **Parameters:** NULL
- **Return:** NULL


~~~ cpp
SDK_API void select_obstacle_distance ( void );
~~~


### set\_image\_frequecy

- **Description:** Set frequency of image transfer.
(**Note:** The bandwidth of USB is limited. If you subscribe too much images (greyscale image or depth image), the frequency should be relatively small, otherwise the SDK cannot guarantee the continuity of image transfer.)
- **Parameters:** `frequency` the frequency of image transfer
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int set_image_frequecy ( e_image_data_frequecy frequecy );
~~~

### select\_depth\_image

- **Description:** Subscribe to depth image data.
- **Parameters:** `camera_pair_index` index of camera pair selected.
- **Return:** `NULL`


~~~ cpp
SDK_API void select_depth_image ( e_vbus_index camera_pair_index );
~~~

### select\_greyscale\_image

- **Description:** Subscribe to rectified grey image data.
- **Parameters:** `camera_pair_index` index of camera pair selected; `is_left` whether the image data selected is left.
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int select_greyscale_image ( e_vbus_index camera_pair_index, bool is_left );
~~~

### set\_sdk\_event\_handler

- **Description:** Set callback, when data from GUIDANCE comes, it will be called by transfer thread.
- **Parameters:** `handler` pointer to callback function.
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int set_sdk_event_handler ( user_call_back handler );
~~~

### start\_transfer

- **Description:** Send message to GUIDANCE to start data transfer.
- **Parameters:** NULL .
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int start_transfer ( void );
~~~

### stop\_transfer

- **Description:** Send message to GUIDANCE to stop data transfer.
- **Parameters:** NULL .
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int stop_transfer ( void );
~~~

### release\_transfer

- **Description:** Release the data transfer thread.
- **Parameters:** NULL .
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int release_transfer ( void );
~~~

### get\_online\_status

- **Description:** Get the online status of GUIDANCE sensors.
- **Parameters:** `online\_status[CAMERA\_PAIR\_NUM]` online status of GUIDANCE sensors
- **Return:** `error code`. Non-zero if error occurs.

~~~ cpp
SDK_API int get_online_status (int online_status[CAMERA_PAIR_NUM] );
~~~

**Notes** : These are only used for USB transfer type. Please reference the protocol of Section 2.1.2 and also the example code of `uart_example` when using UART transfer type.
