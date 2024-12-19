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