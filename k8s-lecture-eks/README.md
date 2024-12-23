### K8s on Rancher Desktop
- Rancher Desktop에서 "Enable Kubernetes"로 로컬에서 k8s 설정
- 로컬에서 Lima VM을 이용해서 k8s 클러스터 생성
- k8s 클러스터는 k3s로 1개의 경랑 노드를 생성하는 것이다

### Service
- 클라이언트 요청을 nodePort(30000) -> 서비스 Port (8080) -> Pod Target Port (8080)로 전달
```yaml
apiVersion: v1
kind: Service

metadata:
  name: spring-service

spec:
  type: NodePort
  selector:
    app: backend-app # Deployment 에 정의된 "backend-app" 레이블을 가진 파드들을 연결
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30000
```
![img.png](resources/images/service.png)



### PVC(Persistent Volume Claim)
- Pod 에서 바로 PV(Persistent Volume)을 접근하는 것이 아니라 PVC를 통해 PV를 접근

![img.png](resources/images/pvc.png)


### Spring Boot <-> Mysql
- env에 my-service를 넣어주면 mysql-service로 매핑 (mysql-service의 nodePort)
```yaml
          env:
            - name: DB_HOST
              value: mysql-service #localhost:30002와 매핑해줌 (nodePort)
```
![connect-spring-mysql.png](resources/images/connect-spring-mysql.png)


### Mysql 외부 Port 막기
- Service Type을 ClusterIP로 설정하면 외부에서 접근 불가능
```yaml
spec:
  type: ClusterIP
  selector:
      app: mysql-db
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      # nodePort  제거

```

- 외부에서 DB 접근해야할 경우, 포트포워드 사용
```shell
$ kubectl port-forward pod/mysql-deployment-7cf6d744-v8nlz 3306:3306
```

![service-cluster-ip.png](resources/images/service-cluster-ip.png)


### k8s에서 ECR 이미지 pull 권한 Secret 생성
- k8s가 AWS ECR의 이미지를 pull 받을 수 있는 Secret 생성
```shell
& kubectl create secret generic regcred --from-file=.dockerconfigjson=/home/ubuntu/.docker/config.json --type=kubernetes.io/dockerconfigjson
```

### EKS Architecture
- 클러스터 = 마스터노드 + 워커노드(N개)
  - 마스터 노드 : 클러스터 전체를 관리하는 서버 (개발자의 kubectl 등의 명령어 받는 서버)
  - 워커 노드 : 실제 파드를 띄우는 서버 (마스터의 명령을 받고 서비스르를 띄운다)
- 서비스(Service)
  - ELB에서 온 요청을 각각의 워커노드로 보내는 역할
  - type: LoadBalancer 로 설정해야함
  - port: 80 (앞에 ELB 포트), targetPort: 8080
- 외부 ELB
  - 사용자들의 요청을 분배하고 서비스로 보내는 역할

![eks-architecture.png](resources/images/eks-architecture.png)

### EKS 생성
- EKS 클러스터를 생성하면 마스터 노드는 생성되지만, 워커 노드(노드 그룹)는 별도로 생성해야함
- EKS 로컬에서 접근
  - EKS 클러스터 생성 후, 로컬에서 접근하려면 kubeconfig 설정 필요
  - 로컬에서 EKS 클러스터가 추가되고, Current도 RancherDesktop(Local)에서 EKS로 업데이트됨
```shell
$ aws eks --region ap-northeast-2 update-kubeconfig --name kube-practice
$ kubectl config get-contexts
CURRENT   NAME                                                            CLUSTER                                                         AUTHINFO                                                        NAMESPACE
*         arn:aws:eks:ap-northeast-2:154920292787:cluster/kube-practice   arn:aws:eks:ap-northeast-2:154920292787:cluster/kube-practice   arn:aws:eks:ap-northeast-2:154920292787:cluster/kube-practice   
          rancher-desktop                                                 rancher-desktop

# EKS -> RancherDesktop로 컨텍스트 변경          
$ kubectl config use-context rancher-desktop
   

```
