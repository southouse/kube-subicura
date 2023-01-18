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
### Deployment
- Deployment는 기본적으로 `ReplicaSet`을 이용
- 버전관리가 가능하여, `rollback`에 용이함
- 배포 전략
  - **RollingUpdate**
    - 버전을 업데이트하면 새로운 ReplicaSet을 생성하여 새로운 버전의 Pod를 하나씩 순차적으로 생성하고 정상적으로 동작한다면, 기존 버전의 Pod를 하나씩 삭제
    - 이전 버전과 새로운 버전이 공존할 수 있음
    - `spec.strategy.rollingUpdate.maxUnavailable` 옵션은 동시에 삭제할 수 있는 파드의 최대 개수 (기본값은 `spec.replicas`의 **25%**)
    - `spec.strategy.rollingUpdate.maxSurge` 옵션은 동시에 생성될 수 있는 파드의 최대 개수 (기본값은 `spec.replicas`의 **25%**)
  - **Blue/Green**
    - 새로운 버전과 이전 버전을 각각 미리 프로비저닝 한 상태에서 트래픽을 교체하는 방법 -> **RollingUpdate**의 단점을 해결
    - 새로운 버전의 Deployment를 만들고, Pod의 엔드포인트에 해당하는 `Service`에 선택된 라벨을 교체
      ```yaml
      # Service - blue(이전 버전)
      apiVersion: v1
      kind: Service
      metadata:
        name: test-service
      spec:
        ports:
          - port: 80
            targetPort: 8080
        selector:
          app: test-pod
          color: blue

      # Service - green(현재 버전)
      apiVersion: v1
      kind: Service
      metadata:
        name: test-service
      spec:
        ports:
          - port: 80
            targetPort: 8080
        selector:
          app: test-pod
          color: green # 현재 버전으로 교체
      ```
  - **Canary**
    - 특정 서버나 소수의 유저에게만 새로운 버전을 배포하고 안전성을 테스트
    - A/B테스트와 오류율 및 성능 모니터링에 유용
    - 새로운 버전의 Deployment를 만들고 Pod의 개수는 이전 버전의 Deployment보다 적게 생성
    - `Service`에서 같은 라벨을 지정하여 트래픽이 이전 버전과 새로운 버전에게 갈 수 있도록 설정하여 진행
    - 새로운 버전에 문제가 없다면, 점차 트래픽의 비율을 전환

### Service (TODO, 아직 NodePort와 LB의 개념이 미숙, 추가로 정리하여 업데이트)
- Pod는 쉽게 생성되고, 삭제되기 때문에 별도의 고정된 도메인 혹은 IP를 가진 엔드포인트를 생성
- 종류
  - **ClusterIP**
    - 클러스터 내부에서 동작
    - 서비스 이름을 내부 도메인 서버에 등록하여 Pod 간 서비스 이름으로 통신 가능
    - `spec.type`을 지정해주지 않으면 기본적으로 ClusterIP로 생성
    - `spec.ports.targetPort`를 지정해주지 않으면 기본적으로 `spec.ports.port`와 동일
  - **NodePort**
    - 클러스터 외부(노드)에서 동작
    - 모든 노드에 포트를 오픈하기 때문에, Pod가 실행되고 있는 노드가 사라졌을 때 접근이 불가능
    - `ClusterIP`의 기능을 기본으로 포함
    - `spec.ports.nodePort`를 지정해주지 않으면 *30000-32768* 포트에서 할당
    - 노드의 IP와 포트를 알아야 연결이 가능
  - **LoadBalancer**
    - 모든 노드를 바라보기 때문에 자동으로 살아 있는 노드에 접근
    - `NodePort`의 기능을 기본으로 포함
- 보통의 웹 애플리케이션에서 80 혹은 443 포트를 사용하고 하나의 포트에서 여러 개의 서비스를 도메인이나 경로에 따라 다르게 연결하기 때문에, `NodePort`와 `LoadBalancer`를 제한적으로 사용

## Day3
### 웹 애플리케이션 배포
- 실습 문제 풀이
  - **web-app/wordpress**
  
    ![Screenshot 2023-01-16 at 12 09 25](https://user-images.githubusercontent.com/35317926/212590634-1f0355fe-6926-4a69-a00b-3c437a496999.png)
    ![Screenshot 2023-01-16 at 12 10 08](https://user-images.githubusercontent.com/35317926/212590644-7326f5c6-9e6a-4cdf-af45-5cc447f7ace6.png)

  - **web-app/vote**

      ![Screenshot 2023-01-16 at 12 08 20](https://user-images.githubusercontent.com/35317926/212590622-dc1e866b-d899-422b-880f-1244b08abb56.png)
      ![Screenshot 2023-01-16 at 12 08 45](https://user-images.githubusercontent.com/35317926/212590631-7bfde9b4-59eb-49b1-ba1f-863dbda9b190.png)

## Day4
### Ingress
