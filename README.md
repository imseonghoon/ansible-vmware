# Automation
## Ansible for VMware Automation

아래의 VMware 설명서와 블로그를 참조하여 구성한 내용에 대한 정리입니다. 

- **목적** : 
1) 엔지니어들이 각 고객사의 환경과 같은 버전 및 구성으로 Nested ESXi 랩 설치가 필요한 상황이 종종 발생하는데 이에 필요한 시간을 단축하고자 합니다.
2) 또한 IaC 도구의 하나인 Ansible을 통해서 관리할 수 있는 사용사례에 대한 인사이트를 제공하고자 합니다.

# VMware Docs

### **ESXi 설치에 사용되는 설치 스크립트**

https://docs.vmware.com/kr/VMware-vSphere/8.0/vsphere-esxi-installation/GUID-51BD0186-50BF-4D0D-8410-79F165918B16.html

# Blog

**Deploy nested ESXi VMs using a Kickstart (KS) script and Ansible Automation**

https://mattadam.com/2024/01/09/deploy-nested-esxi-vms-using-a-kickstart-ks-script-and-ansible-automation/

# **데모에 사용된 소프트웨어 버전**

| Software | version |
| --- | --- |
| vSphere ESXi | 8.0 U3 ( 24022515 ) |
| Ansible | 2.15.12 |
| Ansible Galaxy | 2.15.12 |
| Ansible Python Interpreter | 3.9.18 |
| Jinja | 3.1.4 |
| xorriso | 1.5.4 |

# 구성 완료 후 환경

### 1) ESXi Host (3 EA)

- shim-esxi01.tanzukr.com
- shim-esxi02.tanzukr.com
- shim-esxi03.tanzukr.com       ****

![hosts](https://github.com/user-attachments/assets/d2aa09ea-5cec-4261-9701-ef100649e002)

### 2) VM Folder (1 EA)

![folder](https://github.com/user-attachments/assets/c445a1b4-d4d6-4d76-aa8f-ae6e2bbfa805)

### 3) Datastore

- NETAPP_CA
- NAPP_ISO

![datastore](https://github.com/user-attachments/assets/c9ee5042-d428-435c-9a45-7d64f0d9a7ef)

### 4) Standard Switch 설정

- Physical Adapter (vmnic0, vmnic1)
- Port group
    - Management Network(vmk0)
    - vMotion Network(vmk1)
    - VM Network

![network](https://github.com/user-attachments/assets/79c6e45f-cae3-4c6a-a136-af8a93403d7e)

# 스크립트 파일 구성

```yaml
[iradmin@ansible-master create-esxi-ansible]$ tree
.
├── **VMware-VMvisor-Installer-8.0U3-24022510.x86_64.iso**
├── **ansible.cfg**
├── **inventory**
├── **host_create.yaml**
├── **roles**
│   ├── **cleanup**
│   │   └── tasks
│   │       └── main.yaml
│   └── **deployesxi**
│       ├── tasks
│       │   └── main.yaml
│       └── templates
│           └── **ks.j2**
├── **vars**
│   └── creds.yaml
├── delete_vCenter_settings.yaml
├── VSS_setting.yaml
└── vcenter_setting.yaml
```

### **주요 파일 설명**

- **host_create.yaml :** 두 개의 ansible task로 구성되며 deployesxi 와 cleanup role을 호출하는 ansible playbook
- **roles** : 배포(**deployesxi**) 와 공간 절약을 위해 추출하고 업로드 했던 이미지 파일 삭제(**cleanup**) task
- **vars** : 배포 및 삭제에 필요한 변수들을 저장한 파일
- vcenter_setting.yaml : vCenter Server에서 데이터센터, 클러스터, VM Folder 생성 및 각 호스트에 NFS 데이터스토어(NETAPP_CA, NETAPP_ISO)를 마운트하는 작업 자동화 스크립트
- VSS_setting.yaml : 각 호스트에 Standard Switch 구성을 자동화 하는 스크립트
- delete_vCenter_settings.yaml : 환경 삭제 스크립트

> **ks.j2 :** KS.CFG  설치 스크립트
> 

```bash
# https://docs.vmware.com/kr/VMware-vSphere/8.0/vsphere-esxi-installation/GUID-51BD0186-50BF-4D0D-8410-79F165918B16.html 

# VMware End User License Agreement 동의
vmaccepteula

# root password 설정
rootpw "{{ nested_info_global.password }}"

# local disk에 ESXi 설치
install --firstdisk --overwritevmfs

# Host 네트워크 설정
# network --bootproto=dhcp --device=vmnic0
network --bootproto=static --addvmportgroup=0 --ip={{ nested_info.staticip }} --netmask={{ nested_info_global.staticmask }} --gateway={{ nested_info_global.gateway }} --nameserver={{ nested_info_global.dns }} --hostname={{ nested_info.fqdn }} 

# ESXi host 재부팅
reboot

# busybox 열기 및  and commands 실행
%firstboot --interpreter=busybox

# SSH를 위한 remote ESXi Shell 시작 및 활성화
vim-cmd hostsvc/enable_ssh
vim-cmd hostsvc/start_ssh

# TSM을 위한 ESXi Shell 시작 및 활성화
vim-cmd hostsvc/enable_esx_shell
vim-cmd hostsvc/start_esx_shell

# 라이선스가 있는 경우 자동으로 설치합니다.
# vim-cmd vimsvc/license --set "license-key-here"

# ESXi Shell 경고 무시
esxcli system settings advanced set -o /UserVars/SuppressShellWarning -i 1

# CEIP 비활성화
esxcli system settings advanced set -o /UserVars/HostClientCEIPOptIn -i 2

# DNS servers 구성
esxcli network ip dns server add --server={{ nested_info_global.dns }}

# certificate 재생성
/sbin/generate-certificates

# NTP 구성
esxcli system ntp set -s={{ nested_info_global.ntpserver }}
esxcli system ntp set -e=yes

# ESXi host 재부팅
reboot
```

# **수정해야 할 변수 정의**

> Automation/create-esxi-ansible/vars/creds.yaml
>
