---
title: minikube보다 kind를 써야 하는 이유
categories:
- Kubernetes
feature_text: |
  ## minikube보다 kind를 써야 하는 이유
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---
<style>
	thead td { text-align: center; }
	td { border: 1px solid #444444; }
</style>

본 테스트는 다음과 같은 환경에서 실시했습니다.

+ WSL 2 ubuntu 20.04.6 LTS
+ minikube v1.31.1
+ Kubernetes v1.27.3
+ Docker 24.0.4
+ cilium v1.13.4
+ calico v3.26.1
+ kindnet 1.1.0

minikube에서 mariadb:latest를 Deployment로 설치하고 vi 에디터가 없어 패키지매니저를 통해서 설치를 하려고 했더니 저장소를 찾지 못하는 문제가 있었다.

<img src="/assets/images/Kubernetes/package_update_error.png" width="100%" height="100%"/>

또한, nginx:latest를 Deployment로 2개의 Pod를 설치하고 서로의 웹페이지를 호출하면 대상 Pod도 찾지 못하였다.

<img src="/assets/images/Kubernetes/pod_to_pod_unreachable.png" width="100%" height="100%"/>

그래서 CNI가 문제인가 해서 Calico, Cilium, Kindnet을 순서대로 설치하고 테스트 해보았지만 설치가 되거지 않거나 대상 Pod로 연결되지 못했다.

당연히 패키지매니저가 저장소도 찾지 못하였다.

<img src="/assets/images/Kubernetes/calico_run_container_error.png" width="100%" height="100%"/>
<img src="/assets/images/Kubernetes/cilium_run_container_error.png" width="100%" height="100%"/>
<img src="/assets/images/Kubernetes/kindnet_trying_curl.png" width="100%" height="100%"/>

방법을 바꿔서 kind를 통해서 설치를 해보고 테스트를 해보았다. kind로 클러스터를 구성하고 노드를 설치하면 기본으로 kindnet이 kube-system 네임스페이스로 설치된다.

<img src="/assets/images/Kubernetes/kind_successful.png" width="100%" height="100%"/>

이것만 보아도 로컬 테스트 환경을 할 때 minikube보다 kind가 유리해보였다. 물론 컨테이너 설치에는 minikube가 약 1.5배 정도 빠르다.

이 결과를 보고 minikube에서 kind로 테스트 환경을 바꿀 때 주의할 점이 있다. 특히, minikube에서 싱글노드로 테스트했다면 더 주의깊게 살펴야 한다.  

기존 컨테이너 설정파일에서 로컬 volume을 사용했다면 그대로 사용시에 volumeMounts의 hostPath가 맞지 않아 컨테이너 생성에 실패할 수 있다.  

그런 경우, 해당 링크의 내용을 참조해서 mounts path를 변경하면 컨테이너 생성에 성공한다.

https://kind.sigs.k8s.io/docs/user/configuration/#extra-mounts