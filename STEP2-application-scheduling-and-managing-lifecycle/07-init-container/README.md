# Init Container
* Pod container 실행 전에 초기화 역할을 하는 container이다.
* 완전히 초기화가 진행된 다음에 main container를 실행시킨다.
* Init container가 실패하면, 성공할 때까지 Pod를 반복해서 재시작 한다.
    * restartPolicydp Never를 설정하면 재시작하지 않는다.
    
![](/STEP2-application-scheduling-and-managing-lifecycle/images/07-init-containe1.png)