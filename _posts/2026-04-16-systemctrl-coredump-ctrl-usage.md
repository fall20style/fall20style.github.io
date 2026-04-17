

systemd-coredump 요약

## 🛠️ 주요 기능

* 자동 수집: 프로그램 크래시 시 메모리 상태 자동 저장함.
* 데이터 압축: lz4 등으로 압축해서 용량 아낌.
* 메타데이터: 실행 경로, PID, 시그널 등 상세 정보 같이 기록함.
* 로그 통합: journalctl이랑 연동돼서 확인 편함.


## 📂 저장 및 관리

* 저장소: /var/lib/systemd/coredump/에 보관함.
* 설정: /etc/systemd/coredump.conf에서 용량 조절 가능함.
* 도구: coredumpctl 명령어로 관리함.


## 💡 주요 명령어

* coredumpctl list: 전체 덤프 목록 보여줌.
* coredumpctl info [PID]: 상세 정보 확인용임.
* coredumpctl gdb [PID]: 바로 디버깅.
