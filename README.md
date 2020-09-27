# pyulog

이 프로젝트는 ULog 파일을 파싱해서 변환 및 표시할 수 있는 python package이다. ULog는 self-describing logging 포맷으로 되어 있다. 관련 문서는 [여기](https://dev.px4.io/en/log/ulog_file_format.html)를 참고하자.

제공하는 [command line scripts](#scripts)는 :
- `ulog_info`: ULog 파일의 정보를 표시
- `ulog_messages`: ULog 파일로부터 logged message를 표시
- `ulog_params`: ULog 파일로부터 parameter 추출
- `ulog2csv`: ULog 파일을 CSV 파일로 변환
- `ulog2kml`: ULog 파일을 KML 파일로 변환

## 설치

package 매니저로 설치하기:
```bash
pip install pyulog
```

소스코드로 설치하기:
```bash
python setup.py build install
```

## 개발

해당 code를 설치하기 위해서 다음 명령을 사용하여 쉽게 수정할 수 있다.(이렇게 하면 package를 repo에 대한 link로 설치된다.) :

```bash
pip install -e .
```

## 테스팅

```bash
nosetests -sv
```

or 

```bash
python setup.py test
```

## Code Checking 

```bash
pylint pyulog/*.py
```

<span id="scripts"></span>
## Command Line Scripts

모든 script들은 시스템 어플리케이션으로 설치된다.(Python을 지정하거나 system 경로를 지정하지 않고 터미널에서 바로 실행가능) `-h` flag를 지원하여 도움말을 얻을 수 있다.

아래 섹션에서는 사용법과 sample 출력을 보여준다. ([test/sample.ulg](test/sample.ulg)): 

###  ULog 파일의 정보 표시 (ulog_info)

사용법:
```bash
usage: ulog_info [-h] [-v] file.ulg

ULog 파일의 정보를 표시한다.

위치 arguments:
  file.ulg       ULog input file

옵션 arguments:
  -h, --help     도움말을 보여준다.
  -v, --verbose  실행 상세 내용을 출력
```

예제 출력:
```bash
$ ulog_info sample.ulg
Logging start time: 0:01:52, duration: 0:01:08
Dropouts: count: 4, total duration: 0.1 s, max: 62 ms, mean: 29 ms
Info Messages:
 sys_name: PX4
 time_ref_utc: 0
 ver_hw: AUAV_X21
 ver_sw: fd483321a5cf50ead91164356d15aa474643aa73

Name (multi id, message size in bytes)    number of data points, total bytes
 actuator_controls_0 (0, 48)                 3269     156912
 actuator_outputs (0, 76)                    1311      99636
 commander_state (0, 9)                       678       6102
 control_state (0, 122)                      3268     398696
 cpuload (0, 16)                               69       1104
 ekf2_innovations (0, 140)                   3271     457940
 estimator_status (0, 309)                   1311     405099
 sensor_combined (0, 72)                    17070    1229040
 sensor_preflight (0, 16)                   17072     273152
 telemetry_status (0, 36)                      70       2520
 vehicle_attitude (0, 36)                    6461     232596
 vehicle_attitude_setpoint (0, 55)           3272     179960
 vehicle_local_position (0, 123)              678      83394
 vehicle_rates_setpoint (0, 24)              6448     154752
 vehicle_status (0, 45)                       294      13230
```

### ULog 파일로부터 logged 메시지를 표시 (ulog_messages)

사용법:
```
usage: ulog_messages [-h] file.ulg

Display logged messages from an ULog file

positional arguments:
  file.ulg    ULog input file

optional arguments:
  -h, --help  show this help message and exit
```

예제 출력:
```
ubuntu@ubuntu:~/github/pyulog/test$ ulog_messages sample.ulg
0:02:38 ERROR: [sensors] no barometer found on /dev/baro0 (2)
0:02:42 ERROR: [sensors] no barometer found on /dev/baro0 (2)
0:02:51 ERROR: [sensors] no barometer found on /dev/baro0 (2)
0:02:56 ERROR: [sensors] no barometer found on /dev/baro0 (2)
```

### ULog 파일로부터 parameter 추출 (ulog_params)

사용법:
```
usage: ulog_params [-h] [-d DELIMITER] [-i] [-o] file.ulg [params.txt]

Extract parameters from an ULog file

positional arguments:
  file.ulg              ULog input file
  params.txt            Output filename (default=stdout)

optional arguments:
  -h, --help            show this help message and exit
  -d DELIMITER, --delimiter DELIMITER
                        Use delimiter in CSV (default is ',')
  -i, --initial         Only extract initial parameters
  -o, --octave          Use Octave format
```

예제 출력 (콘솔에):
```
ubuntu@ubuntu:~/github/pyulog/test$ ulog_params sample.ulg
ATT_ACC_COMP,1
ATT_BIAS_MAX,0.0500000007451
ATT_EXT_HDG_M,0
...
VT_OPT_RECOV_EN,0
VT_TYPE,0
VT_WV_LND_EN,0
VT_WV_LTR_EN,0
VT_WV_YAWR_SCL,0.15000000596
```

### ULog를 CSV 파일로 변환 (ulog2csv)

사용법:
```
usage: ulog2csv [-h] [-m MESSAGES] [-d DELIMITER] [-o DIR] file.ulg

Convert ULog to CSV

positional arguments:
  file.ulg              ULog input file

optional arguments:
  -h, --help            show this help message and exit
  -m MESSAGES, --messages MESSAGES
                        Only consider given messages. Must be a comma-
                        separated list of names, like
                        'sensor_combined,vehicle_gps_position'
  -d DELIMITER, --delimiter DELIMITER
                        Use delimiter in CSV (default is ',')
  -o DIR, --output DIR  Output directory (default is same as input file)
```


### ULog 파일을 KML 파일로 변환 (ulog2kml)

> **Note** `simplekml` 모듈이 반드시 설치되어 있어야 한다. 만약 설치되어 있지 않은 경우 다음과 같이 설치한다 :
  ```
  pip install simplekml
  ```

사용법:
```
usage: ulog2kml [-h] [-o OUTPUT_FILENAME] [--topic TOPIC_NAME]
                [--camera-trigger CAMERA_TRIGGER]
                file.ulg

Convert ULog to KML

positional arguments:
  file.ulg              ULog input file

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT_FILENAME, --output OUTPUT_FILENAME
                        output filename
  --topic TOPIC_NAME    topic name with position data
                        (default=vehicle_gps_position)
  --camera-trigger CAMERA_TRIGGER
                        Camera trigger topic name (e.g. camera_capture)
```
