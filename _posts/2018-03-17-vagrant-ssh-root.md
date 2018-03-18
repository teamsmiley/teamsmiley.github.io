# vagrant root access 

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

## 자주 발생되는 문제 

* vagrant vm설치시 아이디 비번인 vagrant/vagrant root/vagrant로 설정된다.

* vagrant 로 만든 vm에 비밀번호로 접속이 안된다.
vagrant ssh 할때 기본적으로 key를 이용해서 접속을 한다. 그러므로 password 접속이 안된다.(centos 6,7)

예) Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)

접속가능하게 하려면 ssh 설정을 바꿔줘야한다. 
```bash
sed  -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;
```

* 루트 접속이 가능하게 하려면 
```bash
sed  -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config;
```

* db서버에 자동 로그인되게  ssh key 를 설정한다.

사용하는 서버에서 공개키를 만든다. 
```
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
```
나온 내용을 복사해둔다. 

접속할 서버에서 
```
ssh-keygen -t rsa
vi ~/.ssh/authorized_keys
```
위에서 복사한 내용을 붙여 넣는다. 

이제 접속할 서버에서 로그아웃 한다.

다시 접속할 서버로 접속해본다. 
 

* command 
```
vagrant ssh-config
```

* vagrant share folder 가 잘 안될때 (mount: unknown filesystem type 'vboxsf')
```
vagrant plugin install vagrant-vbguest
```
이렇게 하고 나서 문제 없이된다.





