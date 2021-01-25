# Kube Config 파일을 사용한 인증

### 직접 curl을 사용하여 요청

* key, cert, cacert 키를 가지고 직접 요청 가능하다.
* 그러나 매번 이 요청을 사용하기에는 무리가 있다.

```
$ curl https://kube-api-server:6443/api/v1/pods\
--key user.key
--cert user.crt
--cacert ca.crt
```