## 인턴 업무 정리 

> 포폴용 중요 업무 (keypoint)만 정리 (사진 등 함께 증적 남기기) 
>
> 개발용 포폴이기 때문에 문제점 > 발견 방안/해결 방안(제안) 발견 형식으로 정리하면 good! 

### NGFW 





---

### TMS

1. **VM 환경에 Rocky OS 설치 + Factory Install**  

* ESXi VM 에서 가상 서버 생성 + Rocky-8.10-x86_64-minimal.iso 로 Rocky 설치 및 기본 설정
* Trouble Shooting : 파티션 설정

  * 문제 : 기존 Centos 설치 당시엔 /boot/efi 파티션이 없어도 정상 설치되었으나 Rocky에서는 해당 파티션 필수 
  * 해결 : Rocky OS 최초 설치 시 /boot/efi 포함 , centos -> migration 실행 시 미포함하여도 정상 동작 확인

![image](https://github.com/user-attachments/assets/7225d156-f603-48ae-808a-cd7263003e16)



* TMS-Plus Factory Install 
  * 제품 시리얼 라이선스 등록 
  * 접속 가능한 IP 설정, 다중접속 허용, 코어파일 설정 등 서버 기본 구성


![image](https://github.com/user-attachments/assets/d4bf78b9-044a-4015-b2e6-c0f7c3c4da96)



* 해당 과정 진행하며, ESXi 서버 구축 및 Rocky/TMS Factory INSTALL 설치 가이드 제작

![image](https://github.com/user-attachments/assets/a6495087-493c-4548-9e90-16cdc194c398)



2. **Docker를 활용한 OS 교체 (Centos -> Rocky)**

* 기존 centos 기반 TMS 사용자가 rocky 기반 TMS로 마이그레이션 할 때 가장 중요한 부분은 데이터 보존(mongodb)이다. 

* 다만 centos와 rocky의 호환되는 mongodb 버전이 다르다. (centos - 3.X , rocky - 4. X) 

* mongodb 3.X 버전을 image화 하여 docker로 컨테이너로 구동, 이후 데이터만 mount 하는 방식을 선택 (mongodb upgrade 후 데이터 복제는 상당한 시간이 소요되어 기각)

* 해당 Docker 기반 마이그레이션 실습을 담당하여 진행

  1.  /data 경로의 MongoDB 로그 데이터 백업
  2.  docker, docker-compose 설치
  3.  mongodb 3.X 이미지 로드 

  * $ docker load -i centos79.tar 

  4. 컨테이너와 DB 데이터의 mount 설정

     $ vi mongo-compse.yml

     volumes:         

     /home1/mongodb:/home1/mongodb         

     /data/mongo_tmp:/data 

  5. Rocky 설치 후 기존 파티션 마운팅

     $ mkdir -p /home1 /backup /data << 기존 설정대로 장치 설정 
     $ mount /dev/sdb2 /home1 
     $ mount /dev/sdb3 /backup
     $ mount /dev/sdb4 /data
     $ vi /etc/fstab
     \* $ blkid /dev/sdb2 명령으로 각 장치의 UUIID 확인 또는 백업해둔 /etc/fstab 정보대로 수정

  6. MongoDB 구동

  * $ docker-compose -f mongo-compose.yml up -d

    ![image](https://github.com/user-attachments/assets/bdaef32d-8fa5-4a4f-bfc7-6b0ba3749bed)



* trouble shooting : start 실행 시 mongod segfault 발생하여 구동 실패 (rocky linux - mongo 3.x) 
  * 원인 : 단순 start 실행 시 docker의 mongodb가 아닌 host의 mongodb가 구동
  * 해결 : docker로 먼저 db 구동 후 , 패키지 실행하는 방법으로 메뉴얼 변경 제안





3. **IXIA 계측기를 활용한 NMS 알람 테스트** 

> SNMP 사양 변경 및 다수 고객사에서 발생하는 NMS 알람에 대한 주요 기능 테스트 담당 

* HTTP/TCP (IAP/WEAP) 연동 : 센서의 하드웨어 정보를 연동 관리 서버에 전달한다. 

  EX) CPU, 메모리, HDD, 팬, 포트 등

* SNMP : (네트워크) 장비의 성능과 핵심 기능의 상타 정보를  모니터링(관제)하고 장애 발생 시 특이점을 보고할 수 있는 프로토콜 (UDP 사용, Port 161)

  * sn_manager : 정보를 수집하는 주체 --> 'NMS' 

  * sn_agent : 정보를 제공하는 주체 --> 'snmpd'

* LINUX에 SNMPD 설정 

  ![image](https://github.com/user-attachments/assets/6b3be105-7195-49eb-bebd-962dde98599c)

* 로직 

  1. 센서에 snmpd 설치하여 관리 서버 network 정보 반영 

  2. 서버에서 SNMP GET 으로 센서 snDaemon 값 수집 
  3. 이전에 저장한 값과 비교하여 '상태(MINOR, MAJOR, CRITICAL)' 값이 달라졌다면 알람 발생

![image](https://github.com/user-attachments/assets/5945adff-fe63-448e-be6d-cb6123daf648)



* 테스트 대상 

  * 데몬 다운 테스트

    ![image](https://github.com/user-attachments/assets/ca5a3d97-1d51-476e-aaa8-c74a840e6584)

  * Packet OverFlow 테스트

  ![image](https://github.com/user-attachments/assets/35cf48a6-cdd9-4586-bbce-c28f5a4f7c2e)

  * Packet Drop 테스트

  * Traffic (Inbound/Outbound) Zero 테스트 (토마호크 계측기도 모두 종료 후 테스트)

  

* IXIA 계측기 활용

  * 사용 이유: 포트의 성능 이상의 트래픽을 유입/중단하여 Overflow/Drop 을 체크해야하는데 토마호크로는 대량의 트래픽 유입 및 ON/OFF 가 번거로움
  * IXIA 계측기를 활용하면 Packet의 형태와 양을 편리하게 조정할 수 있음 

![image](https://github.com/user-attachments/assets/7f99d405-ca6d-4c5d-92e0-026675762854)

* Trouble Shooting 

  * 문제 : Packet Overflow 가 한번 발생한 뒤, 미발생함에도 CLEAR 알람이 발생하지 않고 알람이 계속 반복 발생하는 상황 

  * 원인 : SNMP GET으로 조회한 데이터를 이전 수집 데이터(mongodb에 저장) 과 비교하여 일치할 때 알람을 해제해야하는데, 이 '저장' 데몬의 오동작으로 이전값보다 항상 큰 값이 비교되어 알람 반복 발생 

    ![image](https://github.com/user-attachments/assets/0ccc0742-6801-4865-99e0-c121284718a8)



> 해당 테스트를 위해, 서버실에서 계측기 및 장비 환경 구성 







4. **chrony 모듈 활용한 시간 동기화 개선** 

 Rocky 8.10은 ntpdate를 사용하지 않기에 기존 NTP 서버 - TMS 서버간의 시간 동기화 수행이 실패함 

>  내 역할 : Rocky 8.10과 호환하는 chrony 모듈 제안 후 도입으로 인한 side effect 조사 

* trouble shooting 1 : 최초 INSTALL 시 timezone이 Asia/Seoul로 설정되지 않는 현상 
  * 문제 : INSTALL 스크립트에 Timezone 을 Asia/Seoul로 설정하는 명령어가 정상 적용되지 않음 
    (ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime 2 > /dev/null)
  * 원인 : 이미 심볼릭 링크가 설정되어있는 경우 "failed to create symbolic link '/etc/localtime' : File exist" 오류 발생
  * 해결 : -f 옵션 추가하여 timezone 강제 변경 제안

![image](https://github.com/user-attachments/assets/98c70db6-315e-452d-976e-6dc4d9a24b44)



![image](https://github.com/user-attachments/assets/7846b718-aa8c-42b7-976a-6c2690f107be)



* trouble shooting 2 : 일부 장비에서 서버 시간 동기화가 이루어지지 않는 현상 
  * 문제 : NTP 서버와의 시간동기화가 실패함 
  * 원인 : DNS (nameserver) 가 등록되어있지 않아 도메인 조회에 실패 (time.bora.net)
  * 해결 : NetworkManager 를 활용한 인터페이스(eth0) 설정
    * nmtui를 활용하여 GUI 기반 DNS 서버 설정 --> NetworkManager 재기동하여 /etc/resolv.conf 자동 생성 ![image](https://github.com/user-attachments/assets/4508375f-da1f-425d-8700-a6321ca1f76b)





5. **web Proxy 상태 체크 - 세션, 방화벽, 연결 오류 재현**

> 각각의 Proxy 포트에 대한 상태 체크 재현 방법 강구 

![image](https://github.com/user-attachments/assets/7ea49647-23a7-4d80-9573-e03f80607f82)

* 세션 상태 :netstat -natp 결과

  * Proxy 정책 설정한 port를 process kill 

    ```shell
    $ netstat -antp |grep 50007
      tcp6 0 0 :::50007 :::* LISTEN 44243/node
    $ kill -9 44243 
    ```

* 방화벽 상태 : iptables -nL -t nat 결과

  * iptables 활용해 해당 Proxy 포트에 DNAT, REDIRECT 정책 설정 

    ![image](https://github.com/user-attachments/assets/ef743feb-1b72-4c2f-a172-21ab445fade8)

* 연결 오류 : socket 통신 결과 

  * 통신 불가능한 센서 IP/ Port 를 대상으로 Proxy 정책 등록 



6. **rsyslog 모듈 업그레이드로 syslog 기능 확인 **

> Rocky Linux 환경에서도 업그레이드된 rsyslog 8.2410.0-1.el8 모듈 정상 동작하는지 다양한 네트워크 SYSLOG 연동 테스트

* FW, IPS 장비와 SYSLOG 연동 및 파이프 라인 구축 

![image](https://github.com/user-attachments/assets/dce67cdd-bc6d-4068-889a-b1eb5d7aa8ca)

* 서비스 구동 확인 

![image](https://github.com/user-attachments/assets/b9134faf-27f8-4d87-941d-fa567c56ada5)

* 파이프 통신 확인

![image](https://github.com/user-attachments/assets/752acdbc-1e3e-43c5-8324-e7416fff06a7)



* trouble shooting : 데몬 다운 및 segfault 원인 확인

![image](https://github.com/user-attachments/assets/2ffd71f7-9bcd-447d-a9cc-735829b8d138)





7. **Fiddler를 활용한 IAP 연동 응답값 확인**

> IAP 연동 : 연동 데이터를 HTTP REQUEST/RESPONSE 값을 통해 받는 '실시간 트래픽' 기능 

* 문제 : 해당 기능 실행 중, 데몬 비정상종료 및 error tracelog 급증하는 현상 발생  
* 원인 : Fiddler를 통해 mongod에 저장되는 데이터 확인 (Fiddler : 세션 데이터 확인 툴)
  * IAP 연동 미설정 시 해당 응답값에 대한 권한 거부(403) 발생하는데, 이 권한 거부 HTML 페이지를 데이터로써 그대로 저장함을 발견  
* 해결 : 403 등 오류 응답값은 데이터 저장하지 않고 버림 처리 제안 

![image](https://github.com/user-attachments/assets/32cd4fc2-17e3-4415-a490-b982a045f1e7)



8. **디스크 정보 수집 모듈 (storcli)**

> Rocky Linux, CentOS Linux의 디스크정보 수집 모듈 megacli에서 storcli로 교체 
>
> --> 그로 인해 변경된 기능 검증 

* 주기적으로 실행하는 디스크 수집 스크립트에 storcli 모듈 반영 확인 
  * tpp_monitor_scheduler.py 의 tpshell 명령어 옵션 확인

![image](https://github.com/user-attachments/assets/8816ff81-ce7a-431e-824d-21cf8bbb91fc)

* 실행 로그 확인

  - 명령 실행 시점에 실행 로그(start-end) 가 로깅되어있다.

    ```
    $ cat ext_modules_shell_interface.log
    [2024-11-07 17:24:14,225][297140][tms_shell_interface.py][INFO] (main:196) start process pid = 297140, argc = 7, argv = ['tms_shell_interface.py', '--method', 'manage_tms_operation', '--ext_cmd', 'manage_server_resource', '--detail_cmd', 'stor_cli']
    [2024-11-07 17:24:14,225][297140][tms_config_resource_pattern.py][INFO] (Initialize:138) TMS plus - plain mode
    [2024-11-07 17:24:14,233][297140][sqlmap_interface.py][INFO] (__ParseQueryMap:547) not define timeout, default 5, action = execute, method = post
    [2024-11-07 17:24:14,288][297140][tms_config_resource_pattern.py][INFO] (Initialize:264) server type check - master
    [2024-11-07 17:24:14,292][297140][tms_server_resource_manage_module.py][INFO] (RunModule:26) run tms server resource manage module
    [2024-11-07 17:24:14,293][297140][stor_cli_disk_info_gather_command.py][INFO] (RunStorCliCommand:33) run stor cli command
    [2024-11-07 17:24:14,299][297140][stor_cli_disk_info_gather_command.py][INFO] (__gatherStorCliDiskInfo:138) storcli EOF, exit
    [2024-11-07 17:24:14,636][297140][tms_shell_interface.py][INFO] (main:265) end process pid = 297140
    ```

* mongodb-server_disk_status 확인

  - UI와 일치한다.

    ```
    ........(생략)..........
    {
            "_id" : ObjectId("672c657417b57e7cb54b5d86"),
            "port" : "0",
            "port_status" : "Active",
            "link_speed" : "12.0Gb/s",
            "sas" : "0x56c92bf004e0f402",
            "inquiry_data" : "",
            "eid" : "12",
            "slot_number" : "3",
            "did" : "14",
            "firmware_state" : "Onln",
            "dg" : "1",
            "size" : "1.818",
            "intf" : "TB",
            "med" : "SATA",
            "sed" : "HDD",
            "pi" : "N",
            "sesz" : "N",
            "model" : "512B",
            "sp" : "ST2000NM000A-2J2100",
            "type" : "U",
            "shield_counter" : "0",
            "media_error_count" : "0",
            "other_error_count" : "0",
            "drive_temperature" : "24C (75.20 F)",
            "predictive_failure_count" : "0",
            "s.m.a.r.t_alert_flagged_by_drive" : "No",
            "sn" : "WS10HWJZ",
            "manufacturer_id" : "ATA",
            "model_number" : "ST2000NM000A-2J2100",
            "nand_vendor" : "NA",
            "wwn" : "5000C500E04C6716",
            "firmware_version" : "TN03",
            "raw_size" : "1.819 TB [0xe8e088b0 Sectors]",
            "coerced_size" : "1.818 TB [0xe8d00000 Sectors]",
            "non_coerced_size" : "1.818 TB [0xe8d088b0 Sectors]",
            "device_speed" : "6.0Gb/s",
            "ncq_setting" : "Enabled",
            "write_cache" : "N/A",
            "logical_sector_size" : "512B",
            "physical_sector_size" : "512B",
            "connector_name" : "Port 4 - 7",
            "drive_position" : "DriveGroup:1, Span:0, Row:1",
            "enclosure_position" : "1",
            "connected_port_number" : "1(path0)",
            "sequence_number" : "2",
            "commissioned_spare" : "No",
            "emergency_spare" : "No",
            "last_predictive_failure_event_sequence_number" : "0",
            "successful_diagnostics_completion_on" : "N/A",
            "fde_type" : "None",
            "sed_capable" : "No",
            "sed_enabled" : "No",
            "secured" : "No",
            "cryptographic_erase_capable" : "No",
            "sanitize_support" : "Not supported",
            "locked" : "No",
            "needs_ekm_attention" : "No",
            "pi_eligible" : "No",
            "drive_is_formatted_for_pi" : "No",
            "pi_type" : "No PI",
            "number_of_bytes_of_user_data_in_lba" : "0 KB",
            "certified" : "No",
            "wide_port_capable" : "No",
            "multipath" : "No",
            "device_model" : "WS10HWJZST2000NM000A-2J2100",
            "rptdte" : NumberLong("20241107160001"),
            "skey" : 100
    }
    ........(생략)..........
    ```

  

* trouble shooting 1: docker mongo 환경에서의 수집 오류 

  * 문제 : docker container로 mongodb 구동 시 UI의 CPU 사용률에 이상값  출력되는 현상 

    ![image](https://github.com/user-attachments/assets/6b8b8688-36af-49db-b48a-fc6cb19b0ab2)

  * 원인 : WiredTigerCacheSizeGB 가 1 GB 로 설정되며 발생 

  * 해결 : 250 GB 메모리를 사용하기에 32 GB 할당 제안 (정상값 출력됨)

  ![image](https://github.com/user-attachments/assets/89dfa563-25f6-4a34-a0c3-bee49404f65d) 







* trouble shooting 2 : 디스크 체크 로직상 오류로 Badfile 다수 발생(tps_file_resource_check)

  * 원인 : 수집 시간과 장비 시간 불일치 시 과거 데이터를 badfile로 분류함 
  * 해결 : badfile 최대 개수 유지하도록 스크립트 수정하여 검증 후 방안 제안 

  ![image](https://github.com/user-attachments/assets/2adcd595-92a0-40a8-abc7-938d3ec50438)



9. **TAXII 서버 구축 및 연동 설정** 

> Rule 배포/불러오기 할 TAXII 서버를 TMS 서버와 연동하는 작업 

* docker image 로 빠르게 구축/구동

![image](https://github.com/user-attachments/assets/b596ff19-fc9c-4eef-8978-853aea20c5d2)

* SBL.json 편집하여 TMS 서버와 정책 연동 

  * SBL.json 편집

    ![image](https://github.com/user-attachments/assets/e878c1da-ccdd-4902-9846-3c2aa39dbf7e)

  * curl 명령어로 반영

    ![image](https://github.com/user-attachments/assets/6adf2db5-db1e-4035-bac9-0477bb5ebd95)





10. **다양한 자동화 스크립트 제작** 

* rpm_check.py : 패키지 설치 시 자동 설치되는 RPM 리스트의 버전 및 설치 여부 점검

  ```python
  # -*- coding: utf-8 -*-
  import subprocess
  
  def make_dict():
      # rocky_rpm.txt 파일을 열어서 한 줄씩 처리
      with open('rpm_check.txt', 'r') as file:
          # 결과를 저장할 리스트
          modified_lines = []
  
          # 파일의 각 줄을 읽어서 탭을 쉼표로 변환
          for line in file:
             # print(line.strip())  # .strip()을 사용하여 줄 끝의 개행 문자 제거
              # 공백을 쉼표로 변환하고, 변환된 줄을 리스트에 저장
              modified_line = line.strip().split('\t')  # .strip()으로 개행 제거 후 split
              modified_lines.append(modified_line)
  
      # dictionary 를 하드코딩하지 않고 파일을 읽어 만듬 (파일은 미리 복사붙여넣기로 준비해야함).
      outputs = dict()
      for modified_line in modified_lines:
         # print(modified_line)
          global key
          if len(modified_line) > 2:
              key = modified_line[0]
              outputs[key] = dict()
              outputs[key][modified_line[1]] = modified_line[2]
          elif len(modified_line)==2:
             # print(key)
              outputs[key][modified_line[0]] = modified_line[1]
  
      return outputs
  
  def main(dictRPM):
  
      print("Dictionary keys:", dictRPM.keys())
      strVersionCommand = "yum list installed | grep {} | awk '{{print $2}}' | head -n 1"  # head -n 1은 rocky에서만 추가 (위키에 '.x86_64' 등의 확장자 미표시로 여러 패키지가 동시 조회됨) 
      strLogCommand = "yum list installed {}" 
  
      # 대분류
      for capModule in dictRPM.keys():
          tmpRPM = dictRPM.get(capModule)
  
          # 대분류 내 모듈
          for module in tmpRPM.keys():
              tmpCommand = strVersionCommand.format(module)
              tmpLogCommand = strLogCommand.format(module)
  
              try:
                  # Local version을 가져오기
                  strLocalVersion = subprocess.check_output(tmpCommand, universal_newlines=True, shell=True).strip()
                  strJsonVersion = tmpRPM.get(module)
  
                  # 결과 비교
                  if strLocalVersion == strJsonVersion:
                          strResult = "성공" 
                  else:
                          strResult = "확인 필요" 
  
                  # 대분류 | rpm 모듈명 | 패키지 버전 | Rocky 설치 버전 | 정상 검증 테스트 명령 | 검증 결과
                  strTemplate = "| {} | {} | {} | {} | {} |" 
                  print(strTemplate.format(capModule, module, strJsonVersion, strLocalVersion, strResult))
  
              except subprocess.CalledProcessError as e:
                  # 명령어가 실패했을 경우 처리
                  #print(f"Error running command for {module}: {e}")   # centos에서 지속적으로 실패하는 코드로, 임시 주석 처리함
                  strResult = "에러" 
                  print(strTemplate.format(capModule, module, strJsonVersion, strLocalVersion, strResult))
  
  if __name__ == "__main__":
      dictRPM = make_dict() # 하드코딩이었던 부분 
      main(dictRPM)
  ```

  

* import_random.py : 특정 조건에 맞는 IP 자동 생성 스크립트 

  * 서버-센서 간의 IP 객체 연동 유효값 테스트 중, 다양한 형태 및 개수의 IPv4,  IPv6 네트워크 생성 (최대 10000개)

  ```python
  import random
  
  # 파일 열기
  with open('Z://output.txt', 'w') as f:
      
      group_id = 100
      test_group = 'test_grp1'
      obj_id = 101
      test_obj = 'test_obj1'    
      ip_address = '10.0.5.150'
      subnet_mask = '255.255.255.255'
      
      # 정상 IP (70개)
      for i in range(70):
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      # 특수 IP : 0, 127, 255로 시작 (30개) 
      for i in range(10): 
          ip_address = f"127.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      for i in range(10): 
          ip_address = f"255.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      # -------------------------------------------------------------------------------------------------------------------
      
      # 중복 IP (20개)
      for i in range(10):
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")  
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")    
          
      # IP 범위 초과 / 미만 / 길이 오류 (15개)
      for i in range(5):
          ip_address = f"{random.randint(256, 300)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"  # 범위 초과
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      for i in range(5):
          ip_address = f"-1.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"  # 범위 미만
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      for i in range(5):
          ip_address = f"0.{random.randint(0, 255)}.{random.randint(0, 255)}." 
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
          
      # Prefix 범위 초과 / 미만 / 길이 오류  (15개)
      for i in range(5):
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"  # 유효하지 않은 prefix
          subnet_mask = f"{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}.256"
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      for i in range(5):
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"  # 유효하지 않은 prefix
          subnet_mask  = f"-1.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 254)}"
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      for i in range(5):
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"  # 유효하지 않은 prefix
          subnet_mask = f"255.255.255." 
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")     
      
      
      # IPv6 (15개)
      for i in range(15) : 
          ip_address = ':'.join(f'{random.randint(0, 0xFFFF):04x}' for _ in range(8))
          subnet_mask = '255.255.255.255'
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n") 
          
      # Netmask 대신 Prefix (15개)
      for i in range(15) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"  # 유효하지 않은 prefix
          subnet_mask = '32'
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")   
  
      # 문자 (30개)
      for i in range(5) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.choice(['a', 'B', 'c D'])}"  # 옥텟에 문자 포함
          subnet_mask = '255.255.255.255'
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
          
      for i in range(5) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.choice(['ㄱ', '나', '다 라'])}"  # 옥텟에 문자 포함
          subnet_mask = '255.255.255.255'
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
          
      for i in range(5) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.choice(['!', '@', '# $'])}"  # 옥텟에 문자 포함
          subnet_mask = '255.255.255.255'
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
      
      for i in range(5) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          subnet_mask = f"255.255.255.{random.choice(['a', 'B', 'c D'])}"  # 옥텟에 문자 포함
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
          
      for i in range(5) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          subnet_mask = f"255.255.255.{random.choice(['ㄱ', '나', '다 라'])}"  # 옥텟에 문자 포함
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
          
      for i in range(5) : 
          ip_address = f"{random.randint(1, 223)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
          subnet_mask = f"255.255.255.{random.choice(['!', '@', '# $'])}"  # 옥텟에 문자 포함
          f.write(f"{group_id}|{test_group}|{obj_id}|{test_obj}|{ip_address}|{subnet_mask}||\n")
  ```

  

* 일일 장비 점검 스크립트 (segfault, core 파일 관리)

  ```shell
  #!/bin/bash
  echo "======================================================================="
  echo "Daily Check"
  echo "======================================================================="
  echo "[ 1. dmesg error check ]"
  dmesg -T |egrep -i 'fail|error'
  echo ""
  echo "[ 2. core file check ]"
  find /home1/TMS40/ -name core.[0-9]* -exec ls -rtl {} \;
  find /home1/TMS40/ -name dbg_info* -exec ls -rtl {} \;
  find /data/tms-core/ -name core.[0-9]* -exec ls -rtl {} \;
  echo ""
  echo "[ 3. zombie process check ]"
  ps -ef |grep -v grep | grep defunct
  echo ""
  echo "[ 4. process etime check ]"
  ps -eo pid,etime,cmd |grep -v grep |egrep 'tp_|tpp_|tps_|elas|mongo|node|httpd' |wc -l
  ps -eo pid,etime,cmd |grep -v grep |egrep 'tp_|tpp_|tps_|elas|mongo|node|httpd'
  echo ""
  echo "[ 5. disk check ]"
  df -h
  ```

  





