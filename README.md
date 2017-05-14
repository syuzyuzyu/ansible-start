【Mac】Ansible環境構築



##ansibleインストール

###インストール
pytonのパッケージ管理ツールpipを利用してansibleをインストールします。
pip2と記載しているのはpython2系を利用するためです。

```
pip2 install ansible
~
Successfully installed MarkupSafe-1.0 PyYAML-3.12 ansible-2.3.0.0 asn1crypto-0.22.0 cffi-1.10.0 cryptography-1.8.1 enum34-1.1.6 idna-2.5 ipaddress-1.0.18 jinja2-2.9.6 packaging-16.8 paramiko-2.1.2 pyasn1-0.2.3 pycparser-2.17 pycrypto-2.6.1 pyparsing-2.2.0 six-1.10.0
```
###動作確認
ansibleコマンドを実行して動作確認をしてみます。

```
ansible localhost -m ping                                                                                                                                                                      

 [WARNING]: Host file not found: /etc/ansible/hosts

 [WARNING]: provided hosts list is empty, only localhost is available

localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

##vagrant virtualboxで動作確認
###vagrant virtualboxのインストール

Homebrew Caskがインストールされていない場合、インストールする。
※パッケージ管理した方が良いと思ったため、dmgからはインストールしない

```
brew tap caskroom/cask
```

vagrant, virtualboxのインストール

```
brew cask install virtualbox vagrant
~
virtualbox was successfully installed!
~
vagrant was successfully installed!
```

###vagrantプロジェクト作成してansibleを動作させる

**vagrantプロジェクトの作成**

```
mkdir ansible-sample
cd ansible-sample
vagrant box add bento/centos-7.2
# → 2)virtualboxを選択
vagrant init bento/centos-7.2
vi Vagrantfile

```

ホストから仮想マシンにアクセスするために、
config.vm.network "private_network", ip: "192.168.33.10"のコメントを外します

```Vagrantfile29行目
#config.vm.network "private_network", ip: "192.168.33.10"
↓
config.vm.network "private_network", ip: "192.168.33.10"
```

そして、vagrantを起動させます

```
vagrant up
```


------------
> vagrant upしたところ以下のようなエラーが発生しました
> 「Vagrant was unable to mount VirtualBox shared folders.」
>
> こちらをみて解決しています。ありがとうございます。
> http://qiita.com/ritukiii/items/9933e1497237bdac9500

------------


次に、起動したvagrantの設定を見てみます

```
vagrant ssh-config                                                                                                                                                                             
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/*user-name*/workspace/ansible-sample/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

**Inventoryファイルの作成**
ansibleの接続先サーバの情報ファイル（Inventoryファイル）を作成します。

```
touch hosts
vi hosts
```

hostsの内容を以下のようにします

```
vagrant-machine ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key
```

**hostsを利用してansibleの接続を確認する**

```
ansible all -i hosts -m ping
```

-------------

> **※ansibleでssh接続エラーが出た場合**
> カレントディレクトリにansible.cfgを作成して、以下の記述を追加します。
>
> ```ansible.cfg
> [defaults]
> host_key_checking = False
> ```

--------------


**Playbookの作成と実行**
Playbookはansibleの実行ファイルです。
yaml形式で記述します。

```
touch site.yml
vi site.yml
```

site.ymlに以下のように記載します

```site.yml
---
- name: playbook test
  hosts: all
  tasks:
```

実行

```
ansible-playbook -i hosts site.yml
```

OKと出れば終了です。
