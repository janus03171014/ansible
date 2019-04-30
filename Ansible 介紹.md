# Ansible 介紹
![](https://i.imgur.com/vYkwoNj.png)

## 什麼是 **Ansible** 
#### &emsp; **Ansible** 是以 Python 程式語言所開發的自動化`組態管理工具`，目標是實現基礎建設即程式碼（Infrastructure as Code）的目標，協助開發者快速部署出一致的工作環境。此外，  Ansible 可以用於部署應用程式以及幫助開發者導入持續整合的作業流程。

##### 摘至 [Red Hat 併購 DevOps 新秀 Ansible | iThome](https://https://www.ithome.com.tw/news/99354) 一文。![](https://i.imgur.com/x7e56Y8.jpg)

#### 我認為 Ansbile 是什麼 ? 
1. 它是一個很方便，且快速的自動化部屬工具。
2. 不用在一台一台的幫host或 vm 安裝少的工具。
3. 只需要透過 ssh 和 python 就可以操作。
4. 業界最常見且使用較平凡的工具。

    #### 優點:
    ● 以YAML的格式編寫，語法清楚容易上手。
    ● 只需要透過 ssh 與遠端機器做溝通。
    ● 不需要中間代理伺服器。
    ● Client 端不需要額外安裝任何軟體。

    #### 優點:
    ● 執行時需要等待一段時間。
    ● 必須事先知道 「被控制IP」。
    
***

## Ansible 是怎麼運行 ?
#### &emsp; 在 Ansible 的世界裡，我們會透過 inventory 檔案來定義有哪些 Managed node (被控端)，並藉由 SSH 和 Python 進行溝通。 ![](https://i.imgur.com/aKJyMMH.png)
<!-- 
Control Machine 指的是我們主要會在上面操作 Ansible 的機器，凍仁喜歡用主控端來形容它。它可以是我們平時用的電腦、手機 1 或機房裡的某一台機器，也可以把它想成是一般 Lab 練習裡的 Workstation。

Managed node 則是被 Ansible 操縱的機器，凍仁喜歡用被控端來形容它。在很多的 Lab 練習裡會用 Server 來稱呼它。
-->
#### 安裝 ansible
```bash=
yum install -y epel-release.noarch
    
yum install -y ansible
```
#### 準備 VM
```shell=
vim /vmdsik/shell/create_vm_setting.sh

#!/bin/bash

vmnames="ans1 ans2 ans3 ans4"
......
        virsh define ${xmldir}/${vm}.xml
done

sh /vmdsik/shell/create_vm_setting.sh

virsh start ans?
```

<!-- ```shell=
#!/bin/bash

vmnames="ans1 ans2 ans3 ans4"

[ -f '/vmdisk/hosts' ] && rm -f /vmdisk/hosts
for vmname in ${vmnames}
do
        if virsh list --name --state-running |grep ${vmname} &> /dev/unll ;then # 檢查vm是否運行
                mac=`virsh dumpxml ${vmname} |grep 'mac address' |sed -e "s/.*mac address='\(.*\)'.*/\1/"`
                ip=`arp -n |grep "${mac}" |awk '{print $1}'`
                echo "${vmname} ansible_host=${ip} ansible_port=22" >> /vmdisk/hosts
        fi
done
``` -->

#### 設定 [inventory file](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
<!-- 我們可以先來看一下 ansible.cfg 設定檔  -->
```bash=
cp /etc/ansible/hosts /etc/ansible/hosts.def

vim /etc/ansible/hosts
# ansible_host: 遠端 ssh 主機位置
# ansible_port: 遠端 ssh port
# ansible_user: 遠端 ssh 使用者
# ansible_ssh_private_key_file: 本機 ssh 私鑰檔路徑
# ansible_ssh_pass: 遠端 ssh 密碼 (建建議改私鑰)

localhost ansible_host=127.0.0.1 ansible_port=22

[vm]
ans1 ansible_host=192.168.19.1 ansible_port=22 
ans2 ansible_host=192.168.19.2 ansible_port=22
ans3 ansible_host=192.168.19.3 ansible_port=22
ans4 ansible_host=192.168.19.4 ansible_port=22

```
###### 測試
```bash=
ansible all -m ping
```
***

## Ansible 怎麼 「 玩 」 
#### 我們可以用 Ad-Hoc command 和 Playbook 兩種方式來操作 Ansible
![](https://i.imgur.com/UmBGX64.png)
#### &emsp; 前者是透過一次次簡短的指令來操作，而後者則是把要執行的任務先編寫好，之後再一次執行。兩者的關係就好像 Linux Shell 裡打指令執行和寫成 Shell Script 再執行。
## Playbook 是什麼 ?
#### Playbook 就字面上的意思為「劇本」。我們可以透過事先寫好的劇本 (Playbook) 來讓各個 Managed Node 進行指定的動作 (Plays) 和任務 (Tasks)。
#### 簡單的說就是，Playbooks 就是 Ansible 的腳本(script)。
![](https://i.imgur.com/ba8KjdU.jpg) <br/> 
## [**YAML**](https://zh.wikipedia.org/wiki/YAML) 是什麼 ?
#### 為什麼用 yaml語法 來編些 ? 因為它像 xml 和 json 一樣，但是又比這些語法要來的讓人容易閱讀。
#### **yaml** 的語法結構非常簡單清楚，因為它本身的結構主要依賴階層及縮排，就像 python 。
#### &emsp;&emsp; **plabooks yaml** 的基礎規則:

#### &emsp;&emsp;&emsp; 1. 大小寫區分。 <br/> &emsp;&emsp;&emsp; 2. 已縮排表示階層關係。 <br/> &emsp;&emsp;&emsp; 3. 縮排不可以使用 [tab] 鍵縮排，只允需空白鍵。 <br/> &emsp;&emsp;&emsp; 4. 以 --- 為開頭 (三個 - )

![](https://i.imgur.com/nqyFJ7j.gif)


```yaml=
vim /vmdisk/playbook/cluster.yml

---
- name: Ansible cluster
  hosts: vm
  
  tasks:
  - name: Use yum module
    yum:
      name: "*"
      state: latest
      
  - yum:
      name: "{{ packages }}"
      state: installed
    vars:
      packages:
        - net-tools
        - bash-completion
        - vim-enhanced
        - setroubleshoot
        - policycoreutils-python-2.5-29.el7.x86_64   
```


## **參考資料**
#### 現代 IT 人一定要知道的 Ansible 自動化組態技巧 <br/> https://chusiang.gitbooks.io/automate-with-ansible/content/02.what-is-the-ansible.html

#### 七分鐘掌握 Ansible 核心觀念 <br/> https://school.soft-arch.net/courses/ansible/lectures/655359

#### Docker：使用Shell或Ansible <br/> https://medium.com/@Joachim8675309/docker-using-shell-or-ansible-7cdceb646d3

#### 利用Ansible部署运行Apache(http)的Docker容器 <br/> https://www.linuxidc.com/Linux/2017-11/148296.htm

***
##### Variables <br/> https://chusiang.github.io/ansible-docs-translate/playbooks_variables.html

##### Inventory <br/> https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html

##### Playbooks <br/> https://docs.ansible.com/ansible/latest/user_guide/playbooks.html

##### Ansible的配置檔案 <br/> https://chusiang.github.io/ansible-docs-translate/intro_configuration.html

##### Ansible 基本操作 <br/> https://hackmd.io/udcE6KeDRM-6DfzVDu6i3Q?view