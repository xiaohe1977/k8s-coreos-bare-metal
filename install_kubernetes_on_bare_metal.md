## һ ��ʼ��
### 1. �����￪ʼ��httpscoreos.comkubernetesdocslatestgetting-started.html
### 2. ��¼������ʼֵ
    ETCD_ENDPOINTS     #1. ����httpipport�����ĸ�ʽд  2. ����ip��ETCD����host��ip��ַ��port��Ĭ��ֵ��2379
    POD_NETWORK=10.2.0.016
    SERVICE_IP_RANGE=10.3.0.024
    K8S_SERVICE_IP=10.3.0.1
    DNS_SERVICE_IP=10.3.0.10
### 3. ��etcsystemdsystem�´������������ļ�
�ļ���Ŀ¼�ṹ����


    ��  kubelet.service               #kubelet�����ļ�
    ��  kublet.service.master         #�ڵڶ����k8s��master�ڵ��ʱ���kubelet�������ļ�
    ��  
    ����docker.service.d               #docker��������
    ��      40-flannel.conf           #������flannel������docker�����Ժ��ٿ���
    ��      
    ����etcd2.service.d                #etcd��������
    ��      40-listen-address.conf    #������ETCD������ⲿ���ڲ����ʵ�ַ
    ��      
    ����flanneld.service.d             #flanneld��������
    ��      40-ExecStartPre-symlink.conf    #������flanneld��������ǰ������һ��������
    ��      
    ����multi-user.target.wants
            docker.service
            etcd2.service
            flanneld.service
            kubelet.service
            
�������ݿ��Բο���httpsgithub.comxiaohe1977k8s-coreos-bare-metal

### 4. ����Kubernetes TLS Assets
TLS Assets��Ҫ�������а�ȫ��֤����kubectlִ��ʱ��Ҫ����Ҫ������Ϣ������TLS Assets���̿��Բο����httpscoreos.comkubernetesdocslatestopenssl.html
���Ѿ���������̷�װ��һЩ�ű����ŵ������httpsgithub.comxiaohe1977k8s-coreos-bare-metaltreemastertls
�����õ���Щ�ļ�������ͳһ�ŵ�etckubernetesssl���档
Ȼ��ִ������������ú�Ȩ�ޣ�
$ sudo chmod 600 etckubernetesssl-key.pem  
$ sudo chown rootroot etckubernetesssl-key.pem


### 5. ��������
�ο���httpsgithub.comxiaohe1977k8s-coreos-bare-metaltreemasteretc  ���������޸����������ļ���

- ��������  
        etcflanneloptions.env  
        etcsystemdsystemflanneld.service.d40-ExecStartPre-symlink.conf 
- ����docker  
        etcsystemdsystemdocker.service.d40-flannel.conf
- Create the kubelet Unit  
        etcsystemdsystemkubelet.service
- Set Up the kube-apiserver Pod
        etckubernetesmanifestskube-apiserver.yaml
- Set Up the kube-proxy Pod  
        etckubernetesmanifestskube-proxy.yaml
- Set Up the kube-controller-manager Pod  
        etckubernetesmanifestskube-controller-manager.yaml
- Set Up the kube-scheduler Pod
        etckubernetesmanifestskube-scheduler.yaml
- Set Up the policy-agent Pod  
        etckubernetesmanifestspolicy-agent.yaml

��Ҫע����ǣ�������Calico��صķ���������ԡ�


## �� Deploy Master

### 6. ��������

- $ sudo systemctl daemon-reload  
ע�⣬��ÿ�θ����������ļ�ʱ�����ִ�������������

- ���������������flannel��Ҫ��pod�����Լ�ETCD��������ַ  
    curl -X PUT -d value={Network$POD_NETWORK,Backend{Typevxlan}} $ETCD_SERVERv2keyscoreos.comnetworkconfig

### 7. ����Kubelet  
    $ sudo systemctl start kubelet
    $ sudo systemctl enable kubelet

### 8. ����Namespace
    curl -H Content-Type applicationjson -XPOST -d'{apiVersionv1,kindNamespace,metadata{namekube-system}}' http127.0.0.18080apiv1namespaces

### 9. ����kubelet����Ӧ���Ѿ��������ˣ������������������һ��
 systemctl status kubelet.service  
    ���⣬���б�Ҫ���һ��docker״̬������������  
 docer ps


## �� Deploy Worker Node(s)

### 10. ���Թ��������²��裬���Ǳ�Ҫʱ���Լ�顣
       
    ��Ϊ��5����׼�������²��������ԡ����Ҫ��飬���Բο���������򵼣�   
    httpscoreos.comkubernetesdocslatestdeploy-workers.html
    
-  TLS Assets
-  Networking Configuration
-  Docker Configuration
-  Create the kubelet Unit
-  Set Up the kube-proxy Pod
-  Set Up kubeconfig


### 11. ��������

$ sudo systemctl daemon-reload

$ sudo systemctl start flanneld
$ sudo systemctl start kubelet
$ sudo systemctl start calico-node

$ sudo systemctl enable flanneld
$ sudo systemctl enable kubelet
$ sudo systemctl enable calico-node

### 12. ������

systemctl status kubelet.service

## �� Setting up kubectl

��һ��û���������ֱ�Ӳο�������httpscoreos.comkubernetesdocslatestconfigure-kubectl.html
�������Ҫ��Լʱ�䣬����������   

### 13. ��������kubectl
1) ֱ�ӹ�������kubectl��   
   curl -O httpsgithub.comxiaohe1977k8s-coreos-bare-metalblobmasterkubectl
   
2) ֱ�����ز�ִ������ű���   
   curl -O httpsgithub.comxiaohe1977k8s-coreos-bare-metalblobmastercoreos_scriptsconfigure_cubectl.sh
   bash configure_cubectl.sh
   
3) ������ý��

    $ kubectl get nodes      
    
    NAME            STATUS      AGE  
    127.17.8.101    Ready       21h


## �� Deploy the DNS Add-on


### 15. ����dns.yml   
    �Ѿ����úã�ֱ�����أ�
    curl -O httpsgithub.comxiaohe1977k8s-coreos-bare-metalblobmastercoreos_scriptsdns-addon.yml  

### 16. ����DNS add-on
    $ kubectl create -f dns-addon.yml
    
    ִ���������������������飺   
    kubectl get pods --namespace=kube-system  grep kube-dns-v11
   
### 14. ��ʾ���нڵ�״̬��ִ����������  
    .kubectl get nodes
    .kubectl get pods --namespace=kube-system  grep kube-dns-v11  
    .kubectl describe services  
    systemctl status kubelet.service    
    
    Ϊ�˷��㣬����Ѿ���װ�ɽű���httpsgithub.comxiaohe1977k8s-coreos-bare-metalblobmastercoreos_scriptsshow.sh
    
## ��¼1���ȹ��Ŀ�

### ��1. ����KubeletServiceʧ��

#### ������kubelet
core@core-01 ~ $ sudo systemctl start kubelet

#### �ٲ鿴kubelet״̬
core@core-01 ~ $ sudo systemctl status kubelet

�� kubelet.service
Loaded loaded (etcsystemdsystemkubelet.service; enabled; vendor preset disabled)
Active activating (auto-restart) (Result exit-code) since Tue 2016-05-24 003432 UTC; 936ms ago
Process 1146 ExecStart=usrlibcoreoskubelet-wrapper --api-servers=http127.0.0.18080 --network-plugin-dir= --network-plugin= --registe
Process 1140 ExecStartPre=usrbinmkdir -p etckubernetesmanifests (code=exited, status=0SUCCESS)
Main PID 1146 (code=exited, status=1FAILURE)

May 24 003432 core-01 systemd[1] kubelet.service Main process exited, code=exited, status=1FAILURE
May 24 003432 core-01 systemd[1] kubelet.service Unit entered failed state.
May 24 003432 core-01 systemd[1] kubelet.service Failed with result 'exit-code'.

#### �ٲ鿴Journal��־������bad HTTP status code 404����

sudo journalctl -r -u kubelet.service

    May 24 003651 core-01 systemd[1] kubelet.service Failed with result 'exit-code'.
    May 24 003651 core-01 systemd[1] kubelet.service Unit entered failed state.
    May 24 003651 core-01 systemd[1] kubelet.service Main process exited, code=exited, status=1FAILURE
    May 24 003651 core-01 kubelet-wrapper[1533] run bad HTTP status code 404
    May 24 003649 core-01 kubelet-wrapper[1533] image downloading signature from httpsquay.ioc1aciquay.iocoreoshyperkube0.19.3aci.asc
    May 24 003649 core-01 kubelet-wrapper[1533] image keys already exist for prefix quay.iocoreoshyperkube, not fetching again
    May 24 003649 core-01 kubelet-wrapper[1533] image remote fetching from URL httpsquay.ioc1aciquay.iocoreoshyperkube0.19.3acilinu
    May 24 003647 core-01 kubelet-wrapper[1533] image searching for app image quay.iocoreoshyperkube
    May 24 003647 core-01 kubelet-wrapper[1533] image using image from file usrlib64rktstage1-imagesstage1-fly.aci
    May 24 003647 core-01 systemd[1] Started kubelet.service.
    May 24 003647 core-01 systemd[1] Starting kubelet.service...
    May 24 003647 core-01 systemd[1] Stopped kubelet.service.
    May 24 003647 core-01 systemd[1] kubelet.service Service hold-off time over, scheduling restart.
    May 24 003637 core-01 systemd[1] kubelet.service Failed with result 'exit-code'.
    May 24 003637 core-01 systemd[1] kubelet.service Unit entered failed state.
    May 24 003637 core-01 systemd[1] kubelet.service Main process exited, code=exited, status=1FAILURE
    May 24 003637 core-01 kubelet-wrapper[1491] run bad HTTP status code 404
    May 24 003635 core-01 kubelet-wrapper[1491] image downloading signature from httpsquay.ioc1aciquay.iocoreoshyperkube0.19.3aci.asc
    May 24 003635 core-01 kubelet-wrapper[1491] image keys already exist for prefix quay.iocoreoshyperkube, not fetching again
    May 24 003635 core-01 kubelet-wrapper[1491] image remote fetching from URL httpsquay.ioc1aciquay.iocoreoshyperkube0.19.3acilinu
    May 24 003633 core-01 kubelet-wrapper[1491] image searching for app image quay.iocoreoshyperkube
    May 24 003633 core-01 kubelet-wrapper[1491] image using image from file usrlib64rktstage1-imagesstage1-fly.aci
    May 24 003633 core-01 systemd[1] Started kubelet.service.
    May 24 003633 core-01 systemd[1] Starting kubelet.service...
    May 24 003633 core-01 systemd[1] Stopped kubelet.service.
    May 24 003633 core-01 systemd[1] kubelet.service Service hold-off time over, scheduling restart.
    May 24 003623 core-01 systemd[1] kubelet.service Failed with result 'exit-code'.
    May 24 003623 core-01 systemd[1] kubelet.service Unit entered failed state.

#### ����취
sudo vim etcsystemdsystemkubelet.service
��������һ���޸������ļ���
Environment=KUBELET_VERSION=v1.2.4_coreos.1


### ��2. curl http127.0.0.18080version ����ʧ��

#### ������ʾ��
curl (7) Failed to connect to 127.0.0.1 port 8080 Connection refused

#### �鿴Journal��־��


    '''
    -- Logs begin at Mon 2016-05-23 115659 UTC, end at Tue 2016-05-24 021618 UTC. --
    May 24 021618 core-01 kubelet-wrapper[8402] W0524 021618.529148 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-scheduler-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021618 core-01 kubelet-wrapper[8402] W0524 021618.528742 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-proxy-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021618 core-01 kubelet-wrapper[8402] W0524 021618.528310 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-controller-manager-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021618 core-01 kubelet-wrapper[8402] W0524 021618.527707 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-apiserver-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021613 core-01 kubelet-wrapper[8402] E0524 021613.004607 8402 event.go202] Unable to write event 'Post http127.0.0.18080apiv1namespaceskube-systemevents dial tcp 127.0.0.18080 connection refused' (may retry after sleeping)
    May 24 021610 core-01 kubelet-wrapper[8402] E0524 021610.529897 8402 pod_workers.go138] Error syncing pod 00ae8a278467a8c378c5e1271ad42748, skipping failed to StartContainer for kube-apiserver with CrashLoopBackOff Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)
    May 24 021610 core-01 kubelet-wrapper[8402] I0524 021610.529652 8402 manager.go2050] Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)
    May 24 021610 core-01 kubelet-wrapper[8402] E0524 021610.528207 8402 kubelet.go1764] Failed creating a mirror pod for kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748) Post http127.0.0.18080apiv1namespaceskube-systempods dial tcp 127.0.0.18080 connection refused
    May 24 021608 core-01 kubelet-wrapper[8402] E0524 021608.530450 8402 kubelet.go1764] Failed creating a mirror pod for kube-scheduler-172.17.8.101_kube-system(89bbc6210d1c82fd0ff8bc04a8d1fa17) Post http127.0.0.18080apiv1namespaceskube-systempods dial tcp 127.0.0.18080 connection refused
    May 24 021608 core-01 kubelet-wrapper[8402] E0524 021608.530170 8402 kubelet.go1764] Failed creating a mirror pod for kube-controller-manager-172.17.8.101_kube-system(83c8dad823686562797212f08ef55033) Post http127.0.0.18080apiv1namespaceskube-systempods dial tcp 127.0.0.18080 connection refused
    May 24 021608 core-01 kubelet-wrapper[8402] W0524 021608.529069 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-scheduler-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021608 core-01 kubelet-wrapper[8402] W0524 021608.528732 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-proxy-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021608 core-01 kubelet-wrapper[8402] W0524 021608.528279 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-controller-manager-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021608 core-01 kubelet-wrapper[8402] W0524 021608.527680 8402 manager.go408] Failed to update status for pod _() Get http127.0.0.18080apiv1namespaceskube-systempodskube-apiserver-172.17.8.101 dial tcp 127.0.0.18080 connection refused
    May 24 021603 core-01 kubelet-wrapper[8402] E0524 021603.003577 8402 event.go202] Unable to write event 'Post http127.0.0.18080apiv1namespaceskube-systemevents dial tcp 127.0.0.18080 connection refused' (may retry after sleeping)
    May 24 021558 core-01 kubelet-wrapper[8402] E0524 021558.531227 8402 pod_workers.go138] Error syncing pod 00ae8a278467a8c378c5e1271ad42748, skipping failed to StartContainer for kube-apiserver with CrashLoopBackOff Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)
    May 24 021558 core-01 kubelet-wrapper[8402] I0524 021558.531019 8402 manager.go2050] Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)
''' 

#### ����취��
1���޸�flannel������Ŀ��ENDPOINTS��http
FLANNELD_ETCD_ENDPOINTS=http172.17.8.1012379

2��������flannel���ٷ��ĵ�û��ǿ���������������Ҳͬ������һ�£�  

    sudo systemctl daemon-reload
    sudo systemctl start flanneld
    sudo systemctl enable flanneld
    sudo systemctl restart docker
    sudo systemctl enable docker

3��ȷ��kube-apiserver.yaml���ã�etcd������Ҫ��http    

    vim kube-apiserver.yaml
    etcd-servers=http172.17.8.1012379

4������kubelet  

    sudo systemctl restart kubelet

5���鿴������־

    $ journalctl -r -u kubelet.service  kubeleg.log
    $ vim kubeleg.log
    
 ������ namespaces kube-system not found�ͺð��ˣ�����ȷ��֮ǰ�������Ѿ������������Ҫ��namespaceû�д��������԰��չ����򵼼����������ˡ�


6) ����http127.0.0.18080version   

    �ɹ������ؽ�����£�  
    
    
    $ curl http127.0.0.18080version
    {
    major 1,
    minor 2,
    gitVersion v1.2.3+coreos.0,
    gitCommit c2d31de51299c6239d8b061e63cec4cb4a42480b,
    gitTreeState clean
    }

