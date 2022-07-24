# Environment

## Linux Machine

### 사양

* CPU: [Intel Xeon Processor L5640](https://ark.intel.com/content/www/kr/ko/ark/products/47926/intel-xeon-processor-l5640-12m-cache-2-26-ghz-5-86-gts-intel-qpi.html) (x2)
* RAM: Hynix DDR3 PC3-8500 ECC 16G (x6)
  * 장착중인 RAM 랭크가 4R 이라 혼용이 필요한 경우 랭크가 2R 인 램으로 구매 필요
* M/B: [Supermicro X8DAH+ DS&G](https://www.supermicro.com/products/motherboard/QPI/5500/X8DAH_.cfm)
* VGA: NVIDIA GTX 970 (CUDA Cores: 1664, 4GB GDDR5 RAM)
* SSD:
  * [리눅스 디스크 I/O 성능 테스트하기. (Feat. dd / hdparm)](https://svrforum.com/os/112595)
    * Read/Write 모두 200 MB/s - 보드가 SATA2 까지 지원이라 SATA3 지원 카드 추가 필요
* HDD:

### 운영체제

* O/S: Ubuntu 20.04 LTS

### 운영체제 설치 후 SSH 연결을 위한 패키지 설치

```shell
sudo apt-get install net-tools # ifconfig 명령어를 이용해 머신에 할당된 IP 확인을 위함
sudo apt-get install ssh # SSH 연결을 위함
sudo apt-get install vim # terminal 편집을 위함
```

### Ubuntu 20.04 LTS 네트워크 설정

* Gateway 및 Use Iface 확인

  ```bash
  $ route -n
  Kernel IP routing table
  Destination     Gateway             Genmask         Flags Metric Ref    Use Iface
  0.0.0.0         xxx.xxx.xxx.xxx     0.0.0.0         UG    100    0        0 enpxxx
  ...
  ```

* netplan 을 이용해 static ip 설정
  
  ```bash
  sudo vim /etc/netplan/00-installer-config.yaml
  ```

  ```yaml
  network:
  ethernets:
    enp1s0f0:
      addresses: [xxx.xxx.xxx.xxx/24]
      gateway4: xxx.xxx.xxx.xxx
      nameservers:
        addresses: [8.8.8.8]
  version: 2
  ```

  ```bash
  sudo netplan apply
  ```

* 참고
  * [ubuntu 20.04.3 LTS 고정 IP (Static IP)설정](https://enowy.tistory.com/33)

## Kubernetes Cluster

* host machine은 k8s control
* worker node 를 추가하기 위해 virtual box를 이용하여 2대의 virtual machine 생성
  * 각 머신은 `Bridged Adapter` 로 네트워크 구성
  * 명령어 목록
    * `vboxmanage list vms`
    * `vboxmanage startvm ${VM NAME or UUID} --type headless`
    * `vboxmanage controlvm ${VM NAME or UUID} pause`
    * `vboxmanage controlvm ${VM NAME or UUID} resume`

* 참고
  * [Virtual Box 네트워크 설정 정리](https://takudaddy.tistory.com/352)

### Kubernetes (Master) 설치

* 모두의 MLOps 참고하여 `Kubeadm` 을 사용한 방법으로 클러스터 구성
  > 만약 쿠버네티스의 모든 기능을 사용하고 노드 구성까지 활용하고 싶다면 kubeadm을 권장해 드립니다.
  * [3. Install Prerequisite](https://mlops-for-all.github.io/docs/setup-kubernetes/install-prerequisite/)
  * [4.3. Install Kubernetes - Kubeadm](https://mlops-for-all.github.io/docs/setup-kubernetes/kubernetes-with-kubeadm/)
  * [5. Install Kubernetes Modules](https://mlops-for-all.github.io/docs/setup-kubernetes/install-kubernetes-module/)
  * [6. (Optional) Setup GPU](https://mlops-for-all.github.io/docs/setup-kubernetes/setup-nvidia-gpu/)

```bash
$ kubectl get nodes
NAME        STATUS   ROLES                  AGE     VERSION
lucas-dev   Ready    control-plane,master   3d10h   v1.21.7
```

* Kubenernets 설치시 `sudo kubeadm init --pod-network-cidr=10.244.0.0/16` 로 수행하는 경우 worker 등록 불가
* master에 해당하는 노드에서 다음 명령 실행

  ```bash
  sudo kubeadm reset
  sudo kubeadm init --apiserver-advertise-address=${MASTER_IP}
  ```

  * `kubeadm init` 명령 수행시 worker 노드가 master 노드와 통신하기 위한 token 및 certiciate 를 포함한 `kubeadm join` 명령어를 확인 가능

인증서도 다시 복사

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Kubernetes Worker 추가



#### Worker Node

* 기본적으로 master 노드처럼 `kubeadm, kubelet, kubectl` 까지는 설치가 동일
* `kubeadm join` 명령을 통해 master 노드에 worker 노드를 등록

  ```bash
  sudo kubeadm join ${MASTER_IP}:6443 --token ${TOKEN} --discovery-token-ca-cert-hash sha256:${CERTIFICATE}
  ```

### Kubernetes Client

* master 노드에 `kubectl`을 통한 제어를 위해 인증서 복사

  ```bash
  mkdir -p $HOME/.kube
  scp -p {CLUSTER_USER_ID}@{CLUSTER_IP}:~/.kube/config ~/.kube/config
  ```

* worker 노드가 추가된것을 확인 가능
  > NotReady로 되어 있는 것은 Node를 내려놓은 상태이기 떄문 

  ```bash
  $ kubectl get nodes
  NAME            STATUS     ROLES                  AGE   VERSION
  k8s-worker-01   NotReady   <none>                 15h   v1.21.7
  k8s-worker-02   NotReady   <none>                 15h   v1.21.7
  ${HOST_NAME}    Ready      control-plane,master   15h   v1.21.7
  ```
  