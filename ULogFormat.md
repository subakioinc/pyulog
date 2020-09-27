# ULog 파일 포맷
 * PX4의 내부 상태를 저장하는 파일 포맷
 * 특징
   * self-describing (format과 message type)
   * logging 내용
     * 센서 정보 (IMU, distance sensor, airspeed, ...)
     * 내부 상태 (cpu load, attitude, postion, ...)
     * printf log 메시지
     * 모든 binary 타입에 대해서 Little Endian 사용
## Data 타입
 * 다음과 같은 binary 타입을 사용한다. C언어에 있는 type과 동일하다.

| Type  | Byte 크기  |
|---|---|
| int8_t, uint8_t  | 1  |
| int16_t, uint16_t  | 2  |
| int32_t, uint32_t  | 4  |
| int64_t, uint64_t  | 8  |
| float  | 4  |
| double  | 8  |
| bool, char  | 1  |

 * 추가로 array로도 사용이 가능하다. (예로 float[5])
 * 일반적으로 모든 string(char [length])는 마지막에 '\0'이 포함되지 않으며 비교시 대소문자 구문을 한다.

## File 구조
 * 파일은 3개 구조로 구성되어 있다.
```
----------------------
|       Header       |
----------------------
|    Definitions     |
----------------------
|        Data        |
----------------------
```

## Header 섹션
 * header는 16byte의 고정길이로 되어 있다.
```
----------------------------------------------------------------------
| 0x55 0x4c 0x6f 0x67 0x01 0x12 0x35 | 0x01         | uint64_t       |
| File magic (7B)                    | Version (1B) | Timestamp (8B) |
----------------------------------------------------------------------
```
 * version은 file format에 대한 version을 뜻하며 현재는 1이다.
 * timestamp는 uint64_t 정수이며 microsec로 해당 logging의 시작시간을 뜻한다.

## Definitions 섹션
 * 가변길이며 version 정보, format 정의, 초기 parameter 값을 포한한다.
 * Definitions 섹션과 Data 섹션은 message들의 stream으로 구성되어 있다. 각각은 아래와 같은 header로 시작된다.
 * 종류 : B, F, I, M, P, 
```c++
struct message_header_s {
    uint16_t msg_size;
    uint8_t msg_type
};
```
 * msg_size는 header의 3byte를 제외한 message만의 byte 크기를 뜻한다. 
 * msg_type은 B, F, I, M, P 중에 하나로 컨텐츠를 정의한다.
   * 'B' : Flag bitset message.
   * 'F' : format definition
   * 'I' : information message
   * 'M' : information message multi
   * 'P' : parameter message
```c++
struct ulog_message_flag_bits_s {
    struct message_header_s;
    uint8_t compat_flags[8];
    uint8_t incompat_flags[8];
    uint64_t appended_offsets[3]; ///< file offset(s) for appended data if appending bit is set
};
```
 * 이 message는 header 섹션 바로 뒤에 오는 첫번째 message여야만 한다. 따라서 고정 offset을 가진다.
   * compat_flags :
   * incompat_flags :
   *

## Data 섹션
 * 다음과 같은 message가 Data 섹션에 속한다.
 * 종류 : A, R, D, L, C, S, O, I, M, P
   * 'A': 이름으로 message를 subscribe
   * 'R' : message를 subscribe하지 않음 (현재 사용 안함)
   * 'D' : logged data를 포함
   * 'L' : 로그를 사용한 string message (printf 출력)
   * 'C' : 태그를 가지는 로그를 사용한 string message
   * 'S' : sync message
   * 'O' : dropout으로 표시(logging message 손실)
   * 'I' : 위와 동일
   * 'M' : 위와 동일
   * 'P' : parameter message

   
```c++
struct message_add_logged_s {
    struct message_header_s header;
    uint8_t multi_id;
    uint16_t msg_id;
    char message_name[header.msg_size-3];
};
```
   * multi_id : 동일한 message format은 여러개의 instance를 가질 수 있다. 
https://dev.px4.io/v1.11/en/log/ulog_file_format.html