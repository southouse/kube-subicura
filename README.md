# kube-subicura
> `subicura` 님의 기술 블로그 중 '[쿠버네티스 안내서 - 설치부터 배포까지 <실습편>](https://subicura.com/)'에 대한 학습 기록입니다.

## Day1
### Pod
- `livenessProbe` 옵션은 컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 **컨테이너를 재시작**하여 문제를 해결
- `readinessProbe` 옵션은 컨테이너가 준비되었는지 체크하고 정상적으로 준비되지 않았다면 **Pod으로 들어오는 요청을 제외**
- Pod 안에 여러 개의 컨테이너를 포함할 수 있음

### ReplicaSet
- Pod를 정해진 수만큼 복제하고 관리
- `selector`로 Pod의 label 체크해서 매칭
- 동작 중에 조건이 맞지 않다면 파드를 삭제 후 새로운 파드를 생성
  - `spec.replicas` 개수를 유지하려는 성질
- 쉽게 스케일 아웃이 가능
- 단독으로 사용하는 경우는 없고, `Deployment`를 사용

## Day2