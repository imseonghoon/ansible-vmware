# ubuntu-autoinstall

VMware vSphere 환경에서 Ubuntu Server 22.04 VM을 자동 설치하는 Ansible Playbook입니다.

## Prerequisites
ansible이 설치된 Bastion VM 또는 데스크톱이 우분투 리눅스라고 가정합니다. 만약 Redhat 계열을 사용한다면 apt가 아닌 dnf를 사용하면 됩니다.
다음의 명령을 순차적으로 실행하십시오:

ansible-bation VM 안에서 사용하는 경우, 다음의 명령을 실행하십시오. 

* ```cd ubuntu-autoinstall```
* ```mv /home/iradmin/iso/jammy-live-server-amd64.iso /home/iradmin/{{생성한 디렉토리}}/ansible-vmware/ubuntu-autoinstall/```
* ```pip3 install --upgrade -r ./pip_requirements.txt```
* ```ansible-galaxy collection install --upgrade -r requirements.yml```

ansible-bation VM을 사용하는 환경이 아닐 경우, 다음의 명령을 실행하십시오. 
* ```sudo apt update && sudo apt install python3 python3-pip git p7zip-full xorriso```
* ```git clone https://github.com/rutgerblom/ubuntu-autoinstall.git ~/git/ubuntu-autoinstall```
* ```pip3 install --upgrade -r ~/git/ubuntu-autoinstall/pip_requirements.txt```
* ```ansible-galaxy collection install --upgrade -r ~/git/ubuntu-autoinstall/requirements.yml```

## Usage

변수 파일을 복사하고 환경 및 요구 사항에 맞게 설정을 수정한 후, Ubuntu 서버 배포를 시작하려면 다음 절차를 따르세요:

1. **변수 파일 복사 및 수정**

    `variables_sample.yml` 파일을 복사하고 이름을 `variables.yml`로 변경한 다음, 해당 파일 내의 변수를 환경과 요구 사항에 맞게 수정합니다.

   ```bash
   cp variables_sample.yml variables.yml
   ```

   그런 다음 `variables.yml` 파일을 열고, 각 변수를 해당 환경에 맞게 수정합니다. (예: IP 주소, 사용자 이름, 비밀번호 등)

   ```bash
   vim variables.yml
   ```

   원하는 대로 설정을 수정한 후 파일을 저장하고 닫습니다.

3. **Ubuntu 서버 배포 시작**

   설정이 완료되면, 아래 명령을 사용하여 Ansible Playbook을 실행하여 Ubuntu 서버 배포를 시작합니다.

   ```bash
   ansible-playbook DeployUbuntu.yml
   ```

Playbook이 실행되면 설정된 환경에 맞게 Ubuntu 서버가 자동으로 배포됩니다.
