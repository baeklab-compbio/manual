# SSH 접근 포트 추가

종종 기관이나 단체 네트워크 망으로 SSH 접근에 제약이 있는 경우가 있다. 보안상의 이유로 SSH 접근 시 이용하는 22번 port를 막아 놓는 경우가 있기 때문이다. 따라서 다른 port를 추가로 열어 SSH를 서비스하는 방법을 이용하면 외부에서도 SSH 접근이 가능하다.

## SSH 설정 파일에 포트 추가하고 SSH 데몬 재시작 하기

root 권한을 획득한 후 `/etc/ssh/sshd_config` 파일을 에디터로 열어 포트 설정 부분을 수정한다. 기본으로 `Port 22`라고 되어있는 부분 아래에 `Port 8022`라고 추가적으로 적어주면 22번 포트 말고도 8022번 포트를 통해서도 SSH를 서비스할 수 있다.

```
Port 22
Port 8022
```

SSH 데몬을 다음과 같이 재시작하여 적용시켜주면 된다.

```
# service sshd restart
Stopping sshd:         [  OK  ]
Starting sshd:         [  OK  ]
```

## 닫혀있는 방화벽 포트를 열기 위한 방화벽 설정

SSH 설정으로 특정 port로의 접근을 열어놓았다 하더라도 서버 자체의 port가 닫혀 있다면 무용지물이다. 방화벽 설정을 통해 해당 port도 추가로 열어 주어야 한다.

### 1. root 권한을 이용하여 `/etc/sysconfig/iptables` 파일을 연다. (CentOS 6 기준)

```
-A INPUT -i em1 -p tcp -m tcp --dport 22 -m state --state NEW -j ACCEPT
-A INPUT -i em1 -p tcp -m tcp --dport 972 -m state --state NEW -j ACCEPT
```

--dport 부분이 포트를 지정하는 부분이다. 원래는 윗 줄만 적용되어 있을 테지만, 윗 줄을 그대로 복사하여 22라는 숫자만 972로 바꿔준다. 그러면 22번 포트와 972번 포트를 방화벽에서 허용해줄 수 있다.

(주의) 방화벽 설정에는 정책들의 순서가 중요하다. 따라서 가끔 972번 포트를 열었는데도 외부에서 접속이 불가능할 때가 많다. 따라서 972번 포트를 여는 설정을 해주는 정책을 22번 포트와 비슷한 위치에 놓는 것을 권장한다.

(주의) 만약 파일을 열어 수정한 것이 아니라 `iptables -A ...` 명령을 이용해 정책을 추가하였다면, 방화벽을 다시 시작하기 전에 설정된 내용을 `service iptables save`을 통해 저장해주어야 할 필요가 있다. 이는 명령줄에서 추가한 정책들을 `/etc/sysconfig/iptables` 파일에 저장하는 기능을 한다.

### 2. 방화벽의 변경된 정책 적용을 위해 재시작한다.

```
# service iptables restart
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Setting chains to policy ACCEPT: filter nat      [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]
```

### (추가 정보) 방화벽 관련 명령어들

* 방화벽에 특정 포트(예: 8888) 허용 정책 추가: `iptables -A INPUT -p tcp --dport 8888 -j ACCEPT`
* 방화벽에 현재 설정되어 있는 정책들 확인: `iptables -L` 또는 `iptables --list`
* 명령줄에서 추가한 방화벽 설정 저장: `service iptables save`
* 변경한 설정을 적용하기 위해 방화벽 재시작: `service iptables restart`

### (추가 정보) 열려있는 포트 확인하기

```
netstat -tnlp | grep LISTEN
```

netstat 명령을 이용하면 열려있는 포트를 확인할 수 있다. ssh에서 포트를 추가로 여는 작업을 마쳤다면 제대로 되어있는지를 여기서 확인해볼 수 있다.

## 접속 확인하기

외부 서버에서 다음과 같이 ssh 명령의 -p 옵션을 이용해 다른 포트를 통해 ssh 접속을 시도해 보면 된다.

```
ssh -p 972 147.47.123.456
```

### (추가 정보) 클러스터 시스템 내의 노드 간 SSH 권한 설정

클러스터 시스템 내의 노드들을 왔다갔다 할 필요가 생길 때가 있다. 서버와 노드간 인증 설정이 되어있지 않은 상태에서 SSH 접속을 시도하면 매번 비밀번호를 물어보게 되어 귀찮을 수 있다. 비밀번호를 물어보지 않고 접속할 수 있게 하려면 두 서버간의 인증을 해야 한다. 다 똑같이 해주었는데도 비밀번호를 계속 물어본다면, .ssh 디렉토리 및 authorized_keys 파일의 권한을 살펴볼 필요가 있다. 즉 해당 사용자만 읽고 쓰는 것이 가능해야 실제로 외부에서 비밀번호를 물어보지 않고 들어올 수 있는 ssh를 사용할 수 있다.
