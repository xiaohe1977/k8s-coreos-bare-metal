## һ ��ʼ��

### 1. �����￪ʼ��  

<https://coreos.com/kubernetes/docs/latest/getting-started.html>

### 2. ��¼������ʼֵ
    ETCD_ENDPOINTS     #1.���� `http://ip:port`�����ĸ�ʽд  2.����ip��ETCD����host��ip��ַ��port��Ĭ��ֵ��2379  
    POD_NETWORK=10.2.0.0/16  
    SERVICE_IP_RANGE=10.3.0.0/24  
    K8S_SERVICE_IP=10.3.0.1  
    DNS_SERVICE_IP=10.3.0.10  
    
### 3. ��etc/systemd/system�´������������ļ�
�ļ���Ŀ¼�ṹ����


    ��  kubelet.service               #kubelet�����ļ�
    ��  kubelet.service.master         #�ڵڶ����k8s��master�ڵ��ʱ���kubelet�������ļ�
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
            
�������ݿ��Բο���     

<https://github.com/xiaohe1977/k8s-coreos-bare-metal>

### 4. ����Kubernetes TLS Assets
TLS Assets��Ҫ�������а�ȫ��֤����kubectlִ��ʱ��Ҫ����Ҫ������Ϣ������TLS Assets���̿��Բο����   

<https://coreos.com/kubernetes/docs/latest/openssl.html>

���Ѿ���������̷�װ��һЩ�ű����ŵ�������:   

<https://github.com/xiaohe1977/k8s-coreos-bare-metal/tree/master/tls>  

�����õ���Щ�ļ�������ͳһ�ŵ�etc/kubernetes/ssl���档
Ȼ��ִ������������ú�Ȩ�ޣ�
  
    $ sudo chmod 600 etc/kubernetes/ssl-key.pem  
    $ sudo chown root:root etc/kubernetes/ssl-key.pem


### 5. ��������

�ο���<https://github.com/xiaoe1977/k8s-coreos-bare-metal/tree/master/etc>  ���������޸����������ļ���

- ��������  

        /etc/flannel/options.env
        /etc/systemd/system/flanneld.service.d/40-ExecStartPre-symlink.conf 

- ����docker  
 
        /etc/systemd/system/docker.service.d/40-flannel.conf

- Create the kubelet Unit  
 
        /etc/systemd/system/kubelet.service

- Set Up the kube-apiserver Pod

        /etc/kubernetes/manifests/kube-apiserver.yaml

- Set Up the kube-proxy Pod  

        /etc/kubernetes/manifests/kube-proxy.yaml

- Set Up the kube-controller-manager Pod  

        /etc/kubernetes/manifests/kube-controller-manager.yaml

- Set Up the kube-scheduler Pod

        /etc/kubernetes/manifests/kube-scheduler.yaml

- Set Up the policy-agent Pod  

        /etc/kubernetes/manifests/policy-agent.yaml

**��Ҫע����ǣ���������ò�������Calico��صķ�������á�**


## �� Deploy Master

### 6. ��������

    $ sudo systemctl daemon-reload  

���������������flannel��Ҫ��pod�����Լ�ETCD��������ַ   
    
    $ curl -X PUT -d "value={\"Network\":\"$POD_NETWORK\",\"Backend\":{\"Type\":\"vxlan\"}}" "$ETCD_SERVER/v2/keys/coreos.com/network/config"

### 7. ����Kubelet  
    
    $ sudo systemctl start kubelet
    $ sudo systemctl enable kubelet

### 8. ����Namespace
    
    $ curl -H "Content-Type: application/json" -XPOST -d'{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"calico-system"}}' "http://127.0.0.1:8080/api/v1/namespaces"

### 9. ����kubelet����Ӧ���Ѿ��������ˣ������������������һ��
    
    systemctl status kubelet.service  

���⣬���б�Ҫ���һ��docker״̬������������  
    
    docer ps                       


## �� Deploy Worker Node(s)

### 10. ���Թ��������²��裬���Ǳ�Ҫʱ���Լ�顣
       
��Ϊ��5����׼�������²���������:

    -  TLS Assets
    -  Networking Configuration
    -  Docker Configuration
    -  Create the kubelet Unit
    -  Set Up the kube-proxy Pod
    -  Set Up kubeconfig

���Ҫ��飬���Բο���������򵼣�   

<https://coreos.com/kubernetes/docs/latest/deploy-workers.html>

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

��һ������ֱ�Ӳο�������

<https://coreos.com/kubernetes/docs/latest/configure-kubectl.html>

�������Ҫ��Լʱ�䣬����������   

### 13. ��������kubectl

ֱ�ӹ�������kubectl��   

    curl -O https://github.com/xiaohe1977/k8s-coreos-bare-metal/blob/master/kubectl
   
ֱ�����ز�ִ������ű���   

    curl -O https://github.com/xiaohe1977/k8s-coreos-bare-metal/blob/master/coreos_scripts/configure_cubectl.sh  
    bash configure_cubectl.sh
   
������ý��

    $ kubectl get nodes      
    
    NAME            STATUS      AGE  
    127.17.8.101    Ready       21h


## �� Deploy the DNS Add-on


### 15. ����dns.yml   
    
�Ѿ����úã�ֱ�����أ�
    
    curl -O https://github.com/xiaohe1977/k8s-coreos-bare-metal/blob/master/coreos_scripts/dns-addon.yml  

### 16. ����DNS add-on

    $ kubectl create -f dns-addon.yml
    
ִ���������������������飺   
    
    $ kubectl get pods --namespace=kube-system | grep kube-dns-v11
   
### 17. ��ʾ���нڵ�״̬��ִ����������  

    kubectl get nodes
    kubectl get pods --namespace=kube-system | grep kube-dns-v11  
    kubectl describe services  
    systemctl status kubelet.service    
    
Ϊ�˷��㣬����Ѿ���װ�ɽű���<https://github.com/xiaohe1977/k8s-coreos-bare-metal/blob/master/coreos_scripts/show.sh>



******************

    
## ��¼1���ȹ��Ŀ�


### ��1. ����KubeletServiceʧ��

#### ������kubelet

    core@core-01 ~ $ sudo systemctl start kubelet

#### �ٲ鿴kubelet״̬

    core@core-01 ~ $ sudo systemctl status kubelet

������£�

	kubelet.service
	Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
	Active: activating (auto-restart) (Result: exit-code) since Tue 2016-05-24 00:34:32 UTC; 936ms ago
	Process: 1146 ExecStart=/usr/lib/coreos/kubelet-wrapper --api-servers=http://127.0.0.1:8080 --network-plugin-dir= --network-plugin= --registe
	Process: 1140 ExecStartPre=/usr/bin/mkdir -p /etc/kubernetes/manifests (code=exited, status=0/SUCCESS)
	Main PID: 1146 (code=exited, status=1/FAILURE)

	May 24 00:34:32 core-01 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
	May 24 00:34:32 core-01 systemd[1]: kubelet.service: Unit entered failed state.
	May 24 00:34:32 core-01 systemd[1]: kubelet.service: Failed with result 'exit-code'.

#### �ٲ鿴Journal��־������bad http:// status code 404����

    sudo journalctl -r -u kubelet.service

������£�

	May 24 00:36:51 core-01 systemd[1]: kubelet.service: Failed with result 'exit-code'.
	May 24 00:36:51 core-01 systemd[1]: kubelet.service: Unit entered failed state.
	May 24 00:36:51 core-01 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
	May 24 00:36:51 core-01 kubelet-wrapper[1533]: run: bad HTTP status code: 404
	May 24 00:36:49 core-01 kubelet-wrapper[1533]: image: downloading signature from https://quay.io/c1/aci/quay.io/coreos/hyperkube/0.19.3/aci.asc
	May 24 00:36:49 core-01 kubelet-wrapper[1533]: image: keys already exist for prefix "quay.io/coreos/hyperkube", not fetching again
	May 24 00:36:49 core-01 kubelet-wrapper[1533]: image: remote fetching from URL "https://quay.io/c1/aci/quay.io/coreos/hyperkube/0.19.3/aci/linu
	May 24 00:36:47 core-01 kubelet-wrapper[1533]: image: searching for app image quay.io/coreos/hyperkube
	May 24 00:36:47 core-01 kubelet-wrapper[1533]: image: using image from file /usr/lib64/rkt/stage1-images/stage1-fly.aci
	May 24 00:36:47 core-01 systemd[1]: Started kubelet.service.
	May 24 00:36:47 core-01 systemd[1]: Starting kubelet.service...
	May 24 00:36:47 core-01 systemd[1]: Stopped kubelet.service.
	May 24 00:36:47 core-01 systemd[1]: kubelet.service: Service hold-off time over, scheduling restart.
	May 24 00:36:37 core-01 systemd[1]: kubelet.service: Failed with result 'exit-code'.
	May 24 00:36:37 core-01 systemd[1]: kubelet.service: Unit entered failed state.
	May 24 00:36:37 core-01 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
	May 24 00:36:37 core-01 kubelet-wrapper[1491]: run: bad HTTP status code: 404
	May 24 00:36:35 core-01 kubelet-wrapper[1491]: image: downloading signature from https://quay.io/c1/aci/quay.io/coreos/hyperkube/0.19.3/aci.asc
	May 24 00:36:35 core-01 kubelet-wrapper[1491]: image: keys already exist for prefix "quay.io/coreos/hyperkube", not fetching again
	May 24 00:36:35 core-01 kubelet-wrapper[1491]: image: remote fetching from URL "https://quay.io/c1/aci/quay.io/coreos/hyperkube/0.19.3/aci/linu
	May 24 00:36:33 core-01 kubelet-wrapper[1491]: image: searching for app image quay.io/coreos/hyperkube
	May 24 00:36:33 core-01 kubelet-wrapper[1491]: image: using image from file /usr/lib64/rkt/stage1-images/stage1-fly.aci
	May 24 00:36:33 core-01 systemd[1]: Started kubelet.service.
	May 24 00:36:33 core-01 systemd[1]: Starting kubelet.service...
	May 24 00:36:33 core-01 systemd[1]: Stopped kubelet.service.
	May 24 00:36:33 core-01 systemd[1]: kubelet.service: Service hold-off time over, scheduling restart.
	May 24 00:36:23 core-01 systemd[1]: kubelet.service: Failed with result 'exit-code'.
	May 24 00:36:23 core-01 systemd[1]: kubelet.service: Unit entered failed state.

#### ����취

    sudo vim etc/systemd/system/kubelet.service

��������һ���޸������ļ���
    
    Environment=KUBELET_VERSION=v1.2.4_coreos.1

********************

### ��2. curl http://127.0.0.1/8080/version ����ʧ��

#### ������ʾ��

    curl (7) Failed to connect to 127.0.0.1 port 8080 Connection refused

#### �鿴Journal��־��
	
	$ journalctl -r -u kubelet.service

	-- Logs begin at Mon 2016-05-23 11:56:59 UTC, end at Tue 2016-05-24 02:16:18 UTC. --
	May 24 02:16:18 core-01 kubelet-wrapper[8402]: W0524 02:16:18.529148 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-scheduler-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:18 core-01 kubelet-wrapper[8402]: W0524 02:16:18.528742 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-proxy-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:18 core-01 kubelet-wrapper[8402]: W0524 02:16:18.528310 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-controller-manager-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:18 core-01 kubelet-wrapper[8402]: W0524 02:16:18.527707 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-apiserver-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:13 core-01 kubelet-wrapper[8402]: E0524 02:16:13.004607 8402 event.go:202] Unable to write event: 'Post http://127.0.0.1:8080/api/v1/namespaces/kube-system/events: dial tcp 127.0.0.1:8080: connection refused' (may retry after sleeping)
	May 24 02:16:10 core-01 kubelet-wrapper[8402]: E0524 02:16:10.529897 8402 pod_workers.go:138] Error syncing pod 00ae8a278467a8c378c5e1271ad42748, skipping: failed to "StartContainer" for "kube-apiserver" with CrashLoopBackOff: "Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)"
	May 24 02:16:10 core-01 kubelet-wrapper[8402]: I0524 02:16:10.529652 8402 manager.go:2050] Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)
	May 24 02:16:10 core-01 kubelet-wrapper[8402]: E0524 02:16:10.528207 8402 kubelet.go:1764] Failed creating a mirror pod for "kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)": Post http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:08 core-01 kubelet-wrapper[8402]: E0524 02:16:08.530450 8402 kubelet.go:1764] Failed creating a mirror pod for "kube-scheduler-172.17.8.101_kube-system(89bbc6210d1c82fd0ff8bc04a8d1fa17)": Post http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:08 core-01 kubelet-wrapper[8402]: E0524 02:16:08.530170 8402 kubelet.go:1764] Failed creating a mirror pod for "kube-controller-manager-172.17.8.101_kube-system(83c8dad823686562797212f08ef55033)": Post http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:08 core-01 kubelet-wrapper[8402]: W0524 02:16:08.529069 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-scheduler-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:08 core-01 kubelet-wrapper[8402]: W0524 02:16:08.528732 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-proxy-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:08 core-01 kubelet-wrapper[8402]: W0524 02:16:08.528279 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-controller-manager-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:08 core-01 kubelet-wrapper[8402]: W0524 02:16:08.527680 8402 manager.go:408] Failed to update status for pod "_()": Get http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods/kube-apiserver-172.17.8.101: dial tcp 127.0.0.1:8080: connection refused
	May 24 02:16:03 core-01 kubelet-wrapper[8402]: E0524 02:16:03.003577 8402 event.go:202] Unable to write event: 'Post http://127.0.0.1:8080/api/v1/namespaces/kube-system/events: dial tcp 127.0.0.1:8080: connection refused' (may retry after sleeping)
	May 24 02:15:58 core-01 kubelet-wrapper[8402]: E0524 02:15:58.531227 8402 pod_workers.go:138] Error syncing pod 00ae8a278467a8c378c5e1271ad42748, skipping: failed to "StartContainer" for "kube-apiserver" with CrashLoopBackOff: "Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)"
	May 24 02:15:58 core-01 kubelet-wrapper[8402]: I0524 02:15:58.531019 8402 manager.go:2050] Back-off 5m0s restarting failed container=kube-apiserver pod=kube-apiserver-172.17.8.101_kube-system(00ae8a278467a8c378c5e1271ad42748)


#### ����취��
1���޸�flannel������Ŀ��ENDPOINTS��`http://`

    FLANNELD_ETCD_ENDPOINTS=http://172.17.8.101:2379
    
**�ؼ�����һ����**

2��������flannel���ٷ��ĵ�û��ǿ���������������Ҳͬ������һ�£�  

    sudo systemctl daemon-reload
    sudo systemctl start flanneld
    sudo systemctl enable flanneld
    sudo systemctl restart docker
    sudo systemctl enable docker

3��ȷ��kube-apiserver.yaml���ã�etcd������Ҫ��http

    vim kube-apiserver.yaml
    etcd-servers=http://172.17.8.101:2379

4������kubelet  

    sudo systemctl restart kubelet

5���鿴������־

    $ journalctl -r -u kubelet.service > kubeleg.log
    $ vim kubeleg.log
    
 ������ namespaces kube-system not found �ͺð��ˣ�����ȷ��֮ǰ�������Ѿ������������Ҫ��namespaceû�д��������԰��չ����򵼼����������ˡ�


6) ����: <http://127.0.0.18080/version>

�ɹ������ؽ�����£�  
        
	$ curl "http://127.0.0.1:8080/version"
	{
	"major": "1",
	"minor": "2",
	"gitVersion": "v1.2.3+coreos.0",
	"gitCommit": "c2d31de51299c6239d8b061e63cec4cb4a42480b",
	"gitTreeState": "clean"
	}

