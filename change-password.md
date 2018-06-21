# 서버 비밀번호 변경

마스터 노드(예: sirna1) 에서 root 권한 획득 후 다음과 같이 해당 계정의 비밀번호를 변경 (계정 이름이 `user01` 인 경우)

```
[root@sirna1 ~]# passwd user01
Changing password for user user01.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

변경 후, rocks 클러스터 시스템인 경우 다음과 같이 모든 클러스터에 변경 정보 전달

```
[root@sirna1 ~]# rocks sync users
```
