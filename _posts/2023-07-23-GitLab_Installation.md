---
title:  Kubernetes에서 GitLab 설치 및 메모리 제한 환경에서의 최적화
categories:
- GitLab
feature_text: |
  ## Kubernetes에서 GitLab 설치 및 메모리 제한 환경에서의 최적화
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

### 1. Gitlab 설치

우선은 Docker Hub에서 적당한 gitlab 이미지를 찾습니다.

저의 경우에는 무료 기능은 사용할 수 있지만 추후 Ultimate 제품 구독시에 마이그레이션 걱정이 없도록 gitlab-ee를 선택했습니다.

gitlab-ee를 라이센스 없이 설치하면 무료 기능만 이용할 수 있습니다. (https://docs.gitlab.com/ee/administration/license.html)

<img src="/assets/images/Gitlab/gitlab_dockerhub.png" width="100%" height="100%"/>

주의: 설치용량이 좀 큽니다.

처음에는 alphinlinux/gitlab을 설치하려고 했으나 실패해서 위의 gitlab/gitlab-ee를 선택했습니다.

Gitlab의 시스템 요구사항은 다음과 같습니다.

+ Storage: 2.5GB
+ CPU: 4 core
+ Memory: 4GB

하지만 실제 설치해보면 이것만으로는 충분하지 않습니다.

Gitlab에 많은 제품들이 함께 설치되기 때문입니다.

설치되는 제품들은 다음과 같습니다.

+ PostgresSQL
+ Puma
+ Redis
+ Sidekiq
+ Promethus
+ Gitlab Runner

참고: https://docs.gitlab.com/ee/install/requirements.html

우선 설치를 위해서 yaml 파일을 작성합니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
  template:
    metadata:
      labels:
        app: gitlab
    spec:
      containers:
      - name: gitlab-container
        image: gitlab/gitlab-ee:latest
        ports:
        - containerPort: 22
          protocol: TCP
        - containerPort: 80
          protocol: TCP
        - containerPort: 443
          protocol: TCP
        - containerPort: 5432
          protocol: TCP
        volumeMounts:
        # gitlab 설정파일
        - name: gitlab-config
          mountPath: /etc/gitlab/gitlab.rb
          subPath: gitlab.rb
        - name: gitlab-data-volume
          mountPath: /var/opt/gitlab
        - name: gitlab-logs-volume
          mountPath: /var/log/gitlab
        - name: gitlab-config-volume
          mountPath: /etc/gitlab
        resources:
          limits:
            memory: "4096Mi"
            cpu: "4000m"
      volumes:
        - name: gitlab-config
          configMap:
            name: gitlab-configmap
            items:
              - key: gitlab-config
                path: gitlab.rb
        - name: gitlab-data-volume
          hostPath:
            path: /home/ahnjisoo/gitlab/data
        - name: gitlab-logs-volume
          hostPath:
            path: /home/ahnjisoo/gitlab/logs
        - name: gitlab-config-volume
          hostPath:
            path: /home/ahnjisoo/gitlab/config
```

메모리 부족을 겪지 않으려면 조금 설정을 바꿔야 하기 때문에 ConfigMap을 통해서 설정파일을 작성합니다.

설정은 다음과 같습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-configmap
data:
  gitlab-config: |
    puma['worker_processes'] = 0

    sidekiq['max_concurrency'] = 10

    prometheus_monitoring['enable'] = false

    gitlab_rails['env'] = {
      'MALLOC_CONF' => 'dirty_decay_ms:1000,muzzy_decay_ms:1000'
    }
```

참고: https://docs.gitlab.com/omnibus/settings/memory_constrained_envs.html

위의 설정과 다른 이유는 해당 속성들이 최신 버전에서는 deprecated 되었기 때문입니다.

<img src="/assets/images/Gitlab/gitlab_gitaly.png" width="100%" height="100%"/>

그리고 포트를 열어줄 서비스파일도 작성합니다. 

```yaml
apiVersion: v1 
kind: Service
metadata: 
  name: gitlab-svc-nodeport
spec: 
  ports: 
    - name: gitlab-web-port 
      port: 20080 
      targetPort: 80 
  selector: 
    app: gitlab 
  type: NodePort
```

다음 명령어를 통해서 컨테이너를 배포합니다.

```
kubectl create -f gitlab-configmap.yaml -f gitlab-deployment.yaml -f gitlab-service.yaml
```

주의: Gitlab은 설치가 조금 걸립니다. 저의 경우 3~5분 정도 소요되었고 로그를 통해서 Gitlab과 함께 제공된 제품들의 heartbeat를 체크하고 있다면 설치가 완료된 겁니다.

성공적으로 설치되었는지 확인하기 위해서 gitlab이 설치된 pod 혹은 service의 IP 주소를 확인하여 curl을 통해서 페이지에 접속합니다.

```
# curl [pod 또는 service ip주소]:80
curl 10.101.10.240:20080 # service로 확인
curl 10.244.0.97 # pod로 확인
```

저의 경우에는 minikube로 쿠버네티스 클러스터 환경을 구성했기 때문에 포트포워딩을 설정했습니다.

```
kubectl port-forward svc/gitlab-svc-nodeport 20080:20080 > /dev/null 2>&1 &
```

<img src="/assets/images/Gitlab/gitlab_login.png" width="100%" height="100%"/>

혹시 로그인 중에 412에러나 500에러를 만나신다면 메모리와 cpu 사용량을 확인해주세요.

너무 많이 사용하고 있다면 추가적인 최적화가 필요할 수 있습니다.