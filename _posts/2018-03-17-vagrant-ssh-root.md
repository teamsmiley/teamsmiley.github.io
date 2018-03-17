# vagrant centos root 접속되게 하기 

## vagrant로 두개의 vm을 만들고 provision.sh를 실행하게 한다. 
```bash
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "provision.sh"

  config.vm.define "www" do |www|
    www.vm.box = "centos/6"
  end

  config.vm.define "db" do |db|
    db.vm.box = "centos/6"
  end
end
```

vi provision.sh 

```bash
# root password set
echo -e "yourpassword\nyourpassword" | passwd
# root login allow
sed  -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config;
sed  -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;

reboot;
```
기본적으로 vagrant로 세팅하면 root 비번이 없다..그래서 넣어줘야함. 
ssh관련해서 root가 들어올수 잇게 수정해줘야한다.
sshd를 세팅하고 서비스를 재시작해야하는데 잘안되서 강제 리부팅을 함. 

이제 www로 로그인 한후 ssh로 db에 접속해 보자. 
```
vagrant ssh www
ssh root@db
```
password 물어보고 로그인 가능하면 된다.

# 추가할것 

## ssh key를 등록하는 방법은 없을까?


## vagrant ssh 하면 루트로 바로 접근하고 싶다. 
```bash
Vagrant.configure("2") do |config|
  config.vm.provision "shell", path: "provision.sh"

  config.vm.define "www" do |www|
    www.vm.box = "centos/6"
    www.vm.host_name = "www"
    www.vm.network "private_network", ip: "192.168.30.10"
    #www.ssh.username = "root"
    #www.ssh.password = "vagrant"
    #www.ssh.insert_key = 'true'
  end

  config.vm.define "db" do |db|
    db.vm.box = "centos/6"
    db.vm.host_name = "db"
    db.vm.network "private_network", ip: "192.168.30.20"
    #db.ssh.username = "root"
    #db.ssh.password = "vagrant"
    #db.ssh.insert_key = 'true'
  end
end
```

