---
layout: post
title:  "kubernetes에서 인증서를 갱신해보자"
date:   2022-06-08 12:00:00 +0900
categories: k8s
tags: k8s devops cheatsheet
description: ""
---

## 클러스터 구성 시 인증서의 기간을 정해두기

kubeadm에서의 인증서 세팅 시, 최대 기간은 10년으로 고정된다. 실험삼아 10년 이상의 기간을 넣어도 실제 만들어지는 인증서의 기간은 최대 10년이다. kubernetes에서는 인증서의 갱신 기간을 1년으로 두며, 이 이유는 1년마다 kuberenetes의 업그레이드를 권하기 때문이다.  
아래 예제에서는 초기화를 진행하는 yaml 파일에서, controllerManager의 extraArgs를 통해 인증서의 기한을 10년으로 설정하도록 하였다.

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.12
...
controllerManager:
  extraArgs:
    # 1.19 이전의 경우
    experimental-cluster-signing-duration: 87600h
    # 1.19 이후의 경우
    cluster-signing-duration: 87600h
```

kubernetes cluster를 설정한 후 인증서 내용을 확인해보면 아래와 같다.

```terminal
$ sudo kubeadm alpha certs check-expiration
CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Feb 07, 2032 07:20 UTC   9y                                      no
apiserver                  Feb 07, 2032 07:20 UTC   9y              ca                      no
apiserver-kubelet-client   Feb 07, 2032 07:20 UTC   9y              ca                      no
controller-manager.conf    Feb 07, 2032 07:20 UTC   9y                                      no
front-proxy-client         Feb 07, 2032 07:20 UTC   9y              front-proxy-ca          no
scheduler.conf             Feb 07, 2032 07:20 UTC   9y                                      no
 
CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Feb 07, 2032 07:20 UTC   9y              no
front-proxy-ca          Feb 07, 2032 07:20 UTC   9y              no
```

## 이미 만들어진 클러스터에서 인증서를 갱신하기

현재 아래와 같이 apiserver의 인증서가 1달 후 만료로 되어 있다. 이 인증서를 갱신하려 한다.
아래의 예제에서는 현재 사용중인 kubernetes가 1.18이기 때문에 kubeadm의 cert가 alpha로 들어가 있다.

```terminal
$ sudo kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Feb 07, 2032 07:20 UTC   9y                                      no
apiserver                  Jul 02, 2022 08:53 UTC   29d             ca                      no      
apiserver-kubelet-client   Feb 07, 2032 07:20 UTC   9y              ca                      no
controller-manager.conf    Feb 07, 2032 07:20 UTC   9y                                      no
front-proxy-client         Feb 07, 2032 07:20 UTC   9y              front-proxy-ca          no
scheduler.conf             Feb 07, 2032 07:20 UTC   9y                                      no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Feb 07, 2032 07:20 UTC   9y              no
front-proxy-ca          Feb 07, 2032 07:20 UTC   9y              no
```

아래 스탭에 따라 갱신을 진행한다.

```terminal
// kubeadm을 사용하여 apiserver의 인증서를 갱신한다.
$ sudo kubeadm alpha certs renew apiserver --use-api &
[1] 5894
$ Flag --use-api has been deprecated, certificate renewal from kubeadm using the Kubernetes API is deprecated and will be removed when 'certificates.k8s.io/v1' releases.
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

[certs] Certificate request "kubeadm-cert-kube-apiserver-rxlmd" created

// 새로운 인증서가 생성되었다. 아직 적용하지 않았기 때문에 Condition이 Pending이다.
$ kubectl get csr
NAME                                AGE   SIGNERNAME                     REQUESTOR          CONDITION
kubeadm-cert-kube-apiserver-rxlmd   91s   kubernetes.io/legacy-unknown   kubernetes-admin   Pending

// kubectl을 사용하여 인증서를 적용한다.
$ kubectl certificate approve  kubeadm-cert-kube-apiserver-rxlmd
certificatesigningrequest.certificates.k8s.io/kubeadm-cert-kube-apiserver-rxlmd approved
certificate for serving the Kubernetes API renewed
[1]+  Done                    sudo kubeadm alpha certs renew apiserver --use-api

// 다시 상태를 확인하면 Condition이 Approved,Issued로 바뀌어 있는 걸 볼 수 있다.
$ kubectl get csr
NAME                                AGE    SIGNERNAME                     REQUESTOR          CONDITION
kubeadm-cert-kube-apiserver-rxlmd   2m9s   kubernetes.io/legacy-unknown   kubernetes-admin   Approved,Issued
```

## Reference & Reference

[Kubernetes, kubeadm-cert](https://kubernetes.io/ko/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)

## k8s cheatsheet

```terminal
# patch some resources
kubectl patch svc kubernetes --patch "$(cat apiserver-patch.yaml)"

# make .kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# renew all certs
sudo kubeadm init --config master.yaml phase certs all

```
