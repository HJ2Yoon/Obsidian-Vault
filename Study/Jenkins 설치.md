### GPG Error 해결

해당 링크를 통해 가져오는 GPG Key가 2023년 3월 30일 부로 만료된 Key를 가져오기 때문에 나타나는 현상이다.

유효한 GPG Key를 가져오도록 최신 링크로 수정된 아래의 명령어를 입력하여 해결할 수 있다.

```null
$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo apt-key add -
```

GPG Key : **5BA31D57EF5975CA**  
해당 과정을 통해 2026년 3월 26일까지 유효한 키를 얻을 수 있다.

추가적으로 아래의 명령어를 입력하여 Key 등록을 완료하면 _$ sudo apt-get update_를 정상적으로 수행할 수 있다.

```null
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
```

경우에 따라 기존 GPG Key를 제거하는 과정도 수행이 필요할 수도 있다.

jenkins  포트 8888 (기본 8080)