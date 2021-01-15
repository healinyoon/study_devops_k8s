# Accounts

### Accounts Type

Accounts에는 2가지 타입이 존재한다.

* 사용자를 위한 User Account
* 애플리케이션을 위한 Service Account

# Static Token File

* Account를 만드는 가장 쉬운 방법이다.
* apiserver 서비스를 실행할 때, `--token-auth-file={filename}.csv`를 전달한다.
    * kube-apiserver의 StaticPod를 수정해야 한다. = apiserver를 다시 시작해야 적용된다.
* csv file은 패스워드(token), 아이디(사용자 명), 사용자 UID 정보를 저장한다. 그 외 추가적인 정보를 저장할 수 있다.

> (참고) fiel.csv
```
passwd01, usr01, uid01
passwd02, usr02, uid02
passwd03, usr03, uid03
```


