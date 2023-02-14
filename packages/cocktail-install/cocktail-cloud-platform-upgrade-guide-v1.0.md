# Cocktail Cloud Platform Upgrade Guide v1.0

Author: Anonymous Status: Completed 날짜: December 16, 2022 수정일: February 9, 2023

**칵테일 클라우드 플랫폼은 CLI Tool을 사용하여 설치 및 업데이트를 진행 합니다.**

**CCP 명령어 실행 전에 칵테일 클라우드에서 제공하는 Harbor 레지스트리 서버에 업데이트 관련 Chart , Image 들이 서버에 업로드가 되어야 합니다.**

**아래 목차는 CCP 업데이트 명령어 실행 전에 업로드 하는 방법을 설명합니다.**

### 가이드

### 1. Helm Charts 파일 업로드 방법

칵테일 클라우드 플랫폼에서 사용 되는 기본 프로젝트는 아래와 같습니다.

| Harbor 프로젝트                 | 설명                                                                                | 종류            |
| --------------------------- | --------------------------------------------------------------------------------- | ------------- |
| cocktail                    | 칵테일 클라우드 플랫폼 Helm Chart 를 관리 합니다.                                                 | cocktail      |
| cocktail-addon              | 칵테일에서 사용하는 애드온 Chart 를 관리 합니다.                                                    | addon-manager |
| cocktail-monitoring-stack   |                                                                                   |               |
| cocktail-promtail           |                                                                                   |               |
| ingress-nginx               |                                                                                   |               |
| istio-cni                   |                                                                                   |               |
| istio-egress                |                                                                                   |               |
| istio-ingress               |                                                                                   |               |
| istiod                      |                                                                                   |               |
| kube-prometheus-stack       |                                                                                   |               |
| monitoring-agent            |                                                                                   |               |
|                             |                                                                                   |               |
| cube                        | 칵테일 클라우드 플랫폼에서 사용 하는 모니터링 서비스 관련 Kafka , 로그 , NFS Driver , GPU 관련 Chart 를 관리 합니다. | cocktail-log  |
| cocktail-message-queue      |                                                                                   |               |
| cocktail-message-queue-user |                                                                                   |               |
| csi-driver-nfs              |                                                                                   |               |
| nvidia-gpu                  |                                                                                   |               |

💡 위 표 참고하여 관련 프로젝트에 파일을 업로드 하시면 됩니다.

> **주의 - Chart 업로드 후 설치 VM 에서 helm repo update 명령어를 실행 하여 정상적으로 업로드가 되었는지 확인 합니다.**

### 2. 전달 받은 Tar 파일을 Image 로 변경하여 Harbor 서버에 Push

Harbor 주소 예시 - [https://example-harbor.acloud.run](https://hcapital-harbor.acloud.run/)

1. Docker 설치 된 (Harbor) VM 에서 Docker Login

```bash
## 아래 명령어로 아이디/비밀번호를 입력하여 로그인
$ docker login [https://example-harbor.acloud.run](https://hcapital-harbor.acloud.run/)
Username: admin
...
```

1. 다운로드 받은 Tar 파일을 Image 로 Load

```bash

### 예시 파일 - api-server-4.7.3-release.20221214.tar

# ls -al
...
-rw-r--r--    1 nobody   nobody   420734976 Dec 14 02:23 api-cmdb-4.7.3-release.20221201.tar
-rw-r--r--    1 nobody   nobody   593793024 Dec 14 03:56 api-server-4.7.3-release.20221214.tar
...

### tar -> image 변경

$ docker load -i api-server-4.7.3-release.20221214.tar
Loaded image: regi.acloud.run/library/api-server:4.7.3-release.20221214

### 이미지 목록 확인
$ docker images

```

1. Load 한 이미지를 Push 할 Harbor 주소로 변경 및 이미지 Push

```bash
### image tag 
$ docker tag regi.acloud.run/library/api-server:4.7.2-release.20221117 [example-harbor.acloud.run](https://hcapital-harbor.acloud.run/)/library/api-server:4.7.2-release.20221117

### 이미지 확인
$ docker image 

### 이미지 Push
$ docker push [example-harbor.acloud.run](https://hcapital-harbor.acloud.run/)/library/api-server:4.7.2-release.20221117
```

Chart 업로드 및 Image Push , CCP CLI 업데이트 작업을 완료 하면 업그레이드 할 준비 작업은 완료 되었습니다.

### 3. CCP Tool 을 이용하여 칵테일 업그레이드

```bash
## Cocktail 업그레이드 명령어 실행
## Loki , Kafka 변경시 force 옵션 필요 --force
$ ccp upgrade 

## 명령어로 이미지 파드 정상 여부 확인
$ kubectl get po -n cocktail-system -owide 

```

### 4. 칵테일 애드온 Chart 업그레이드 및 Version 확인 방법

* 애드온 목록에서 업로드한 Chart 버젼과 화면에 보이는 버젼이 동일 한지 확인 합니다.
  * 업로드 Version : **1.15.12**
* 애드온 화면에서 Version 노출 확인 : **1.15.12**
  * 화면 깨짐이 없는지 확인
* 애드온 화면에서 Version : 1.15.12 선택 후 저장 버튼을 클릭하여 배포 합니다.
