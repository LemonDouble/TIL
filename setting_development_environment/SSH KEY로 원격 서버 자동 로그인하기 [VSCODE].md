# SSH KEY로 원격 서버 자동 로그인하기 [VSCODE]

배운 내용 (한줄 요약): 비밀번호 없이 자동으로 로그인 하는 방법
분류: 개발환경 설정
자료 링크: https://opentutorials.org/module/432/3742
작성일시: 2021/06/24, 22:03

## 1. ssh-keygen으로 ssh key 만들기

```jsx
$ ssh-keygen -t rsa
```

저장 위치 → password → password 확인 순으로 확인 창이 뜨는데, 기본 위치 / password 없음 / password 없음으로 한다.

## 2. 만들어진 Key 확인 및 키 등록

```jsx
$ ls -al ~/.ssh/
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

id_rsa, id_rsa.pub 가 존재하는지 확인한다.

이 때,

id_rsa : Secret key

id_rsa.pub : Public key

cat : 해당 File 내용을 출력

>, >> : Pipeline, >는 파일을 삭제 후 새로 작성, >>는 뒤에 concat한다.

## 3. ~/.ssh 폴더 권한 설정 (optional)

```jsx
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub  
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
```

authorized_keys 없을 수도 있다.

이때 chmod:

User / Group / Other 순서이고, 

각 단에선 Read / Write / Execute 순서.

따라서 700 : rwx : User만 Read / Write / Execute 가능

644 : rw / r / r : User만 Read / Write 가능, Group과 Others는 Read만 가능

## 4. vscode 설정

id_rsa를 다운로드 하고, 적절한 위치로 이동시킨다.

VScode에서 F1 → Remote ssh : open ssh configuration File

```jsx
Host Lemon-pi
    HostName 192.168.1.143
    User pi
    port 22
    IdentityFile C:\lemonServer\id_rsa << 와 같이 추가해준다.
```

## 5. 윈도우에서 key permission 설정

(Warning : Unprotected Private Key File 뜰 경우)

![https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled.png)

ssh key 있는 곳으로 가 설정 → 보안 → 고급

![https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled%201.png)

상속 사용 안 함 → 이 개체에서 상속된 사용 권한을 모두 제거

![https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled%202.png)

이후 추가(D) 누른 후, 이메일 입력 후 이름 확인, 이후 확인

![https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled%203.png](https://github.com/LemonDouble/TIL/blob/main/setting_development_environment/img/Untitled%203.png)

이후 접속시 정상작동 됨을 확인할 수 있다.
