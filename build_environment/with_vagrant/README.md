# 以 Vagrant 建立 k8s 環境
以 Vagrant 建立虛擬叢集環境，讓 k8s 架設在此虛擬環境之上。

## 環境簡介
Hostname|OS|IP|
---|---|---|
master|Ubuntu 16.04|172.25.0.10|
node1|Ubuntu 16.04|172.25.0.11|
node2|Ubuntu 16.04|172.25.0.12|
node3|Ubuntu 16.04|172.25.0.13|

## Vagrantfile 規劃

### Overview
```ruby
$create_hosts = <<SCRIPT
echo "127.0.0.1 localhost\n\n172.25.0.10 master\n172.25.0.11 node1\n172.25.0.12 node2\n172.25.0.13 node3\n" > /etc/hosts
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = "bento/ubuntu-16.04"
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "172.25.0.10"
    master.vm.provision "shell", inline: $create_hosts
  end

  (1..3).each do |i|
	config.vm.define "node-#{i}" do |node|
	  node.vm.box = "bento/ubuntu-16.04"
	  node.vm.hostname = "node#{i}"
	  node.vm.network "private_network" , ip: "172.25.0.1#{i}"
	  node.vm.provision "shell",
		inline: $create_hosts
    end
  end
end
```

### Vagrant 版本設定

目前能設定的版本只有兩種，分別為：

* 1：For 1.0.x. 
* 2：For 1.1+ leading up to 2.0.x

```ruby
...
Vagrant.configure("2") do |config|
...
```

### 建立 master

在 master 的部分：

* 使用 `Ubuntu 16.04`。
* 使用內部網路，並給予 IP：`172.25.0.10`。
* 設定`/etc/hosts`：讓節點間依設定的 IP 連結。

```ruby
...
  config.vm.define "master" do |master|
      master.vm.box = "bento/ubuntu-16.04"
      master.vm.hostname = "master"
      master.vm.network "private_network", ip: "172.25.0.10"
      master.vm.provision "shell", inline: $create_hosts
    end
...
```

### 建立 node

在 node 的部分：

* 使用 `Ubuntu 16.04`。
* 使用內部網路，並給予 IP：`172.25.0.11-13`。
* 設定`/etc/hosts`：讓節點間依設定的 IP 連結。

```ruby
...
  (1..3).each do |i|
	config.vm.define "node-#{i}" do |node|
	  node.vm.box = "bento/ubuntu-16.04"
	  node.vm.hostname = "node#{i}"
	  node.vm.network "private_network" , ip: "172.25.0.1#{i}"
	  node.vm.provision "shell",
		inline: $create_hosts
    end
  end
...
```

### create_hosts

建立腳本，讓節點在建立時，即可修改節點中的`/etc/hosts`成我們設定的環境需求。

```ruby
$create_hosts = <<SCRIPT
echo "127.0.0.1 localhost\n\n172.25.0.10 master\n172.25.0.11 node1\n172.25.0.12 node2\n172.25.0.13 node3\n" > /etc/hosts
SCRIPT
```