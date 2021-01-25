# Kube Config 파일을 사용한 인증

### (Kube Config 파일을 사용하지 않고도) 직접 curl을 사용하여 요청 가능하다.

* key, cert, cacert 키를 가지고 직접 요청 가능하다.
* **그러나 매번 이 요청을 사용하기에는 무리가 있다.**

```
$ curl https://kube-api-server:6443/api/v1/pods\
--key user.key
--cert user.crt
--cacert ca.crt
```

### Kube Config 파일을 사용한 인증

위의 curl을 사용하는 방법은 불편하므로, Kube Config View에 적용해서 사용한다.

```
$ kubectl config view --kube config={config file}
```

Kube Config Veiw는 크게 **clusters**, **context**, **users** 3가지 부분으로 나누어져 있다. 추가로, 중간에 **current-context** 정보가 포함되어 있는데 이는 `use context`로 선택해두었던 context가 등록되어 있다. 