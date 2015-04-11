---
layout: post
title: Ubuntu14.04でOpenstack
---
 * OpenStack本にしたがってやったらあっさりハマる。まぁ本の名誉のためにいっておくと直接は本のせいじゃない。
 * P79にあるようにubuntu-virtinst.gitをもってくる（成功）。  
 # git clone https://github.com/josug-book1-materials/ubuntu-virtinst.git  
 * openstack用のネットワークを構築する（成功）。  
  # virsh net-define openstack.xml  
  # virsh net-start openstack  
  # virsh net-autostart openstack  
 * コントローラ用のノードを作るためのスクリプトをubuntu-virtinstall.gitからもってきたvirtinst-stack01.shを実行する。すると以下のように出てエラーになる。  
  ~/ubuntu-virtinst# ./virtinst-stack01.sh  
  
  Starting install...   
  Retrieving file MANIFEST...                              | 2.1 kB     00:00 ...  
  Retrieving file MANIFEST...                              | 2.1 kB     00:00 ...  
  Retrieving file linux...                                 |  11 MB     00:02 ...  
  Retrieving file initrd.gz...                             |  41 MB     00:05 ...  
  ERROR    internal error: process exited while connecting to monitor: qemu-system-x86_64: -chardev   
  
  socket,id=charmonitor,path=/var/lib/libvirt/qemu/stack01.monitor,server,nowait: Failed to bind socket: Permission denied
  qemu-system-x86_64: -chardev socket,id=charmonitor,path=/var/lib/libvirt/qemu/stack01.monitor,server,nowait: chardev: opening backend 
    
  "socket" failed  
    
  Domain installation does not appear to have been successful.  
  If it was, you can restart your domain by running:  
    virsh --connect qemu:///system start stack01  
  otherwise, please restart your installation.  
 * ぐぐると[libvirt does not grant qemu-guest-agent channel perms](https://bugs.launchpad.net/ubuntu/+source/libvirt/+bug/1393842)が出てくる。初出2014/11。
 * 内容はlibvirtのバグのようなのだが、直ったのが2015/04/09!（一昨日）どう見ても本出た後だ。
 * いちおう念のためapt-get update && apt-get upgradeはかけてみたが状況変わらず（パッケージの入れ替わりなし）。
 * というか直った扱いになってるのはvividであって、trusty(14.04)じゃないので当たり前といえば当たり前だが。
 * 直ってるとされるlibvirt-1.2.12を野良ビルドしようかとも考えたが、影響範囲が予想つかなすぎて見送り。たった2ページ分しか進まないとかハードル高すぎだろう。
 * と思っていたのだが、犯人は実は別にいた。
 * 犯人はapparmor、お前だ。SELinuxまでは想像できてもお前は想像できなかったよ。
 * /etc/default/grubに以下を設定。  
   GRUB_CMDLINE_LINUX="apparmor=0"  
 * その上で反映。  
   # update-grub  
   # reboot  
 * これでとりあえず仮想マシンの起動そのものはできるようになった。が、Ubuntuをインストールする際にUbuntu archive errorが出てしまいまだ止まっているのだった。
 * どうも内部的にはDHCPでIPアドレスをとっているようだが、その際に渡されたDNSサーバのアドレスのIPのサーバが名前解決をしてくれないようだ。shellに落ちて確認した。
 * 仕方ないのでshellに落ちた状態でecho "nameserver 8.8.8.8" >>/etc/resolv.confを実行してネームサーバを追加。これでUbuntuのインストールは始まったのだが、なんかいろいろはまりすぎだろう。
 * とりあえずインストールは完了したが、これを2台以降もやる気はしないので、ネットワークの設定を修正する。

 ```
  <dns>  
     <txt name="example" value="example">  
     <forwarder addr="8.8.8.8"/>  
     <forwarder addr="8.8.4.4"/>  
  </dns>   
```

* これでまずインストールで止まるということはなくなる、はず。
* 次にやるべきことは同じOSをインストールするのに別々にネットワークに見に行くのは無駄が多いのでProxyを通す。ただし、virsh-installで自動化させたいので手動でProxyを設定するのはNG。ということでtransparent proxy。
* まずsquidの設定。

```
http_port 8080 
http_port 3128 transparent

acl openstack src 192.168.100.0/24
delay_pools 1
delay_calss 1 1
delay_access 1 allow openstack
delay_parameters 300000/300000 # 100kB/s
```

 * 家で通常のプロキシを設定しているPCもあるので、無条件にtransparentにはしない（あちこち直して回るのがめんどい）。3128を別途transparent用のポートとして設定する。ついでに帯域制限もかけた。今のところopenstackの環境にネットワーク処理速度を要求することはないし、自分が本筋でやりたいことを邪魔されたくないので。
 * 次にiptablesの設定。

```
iptables -t nat -A PREROUTING -s 192.168.100.0/24 -p tcp --dport 80 -j REDIRECT --to-port 3128
iptables -t nat -A PREROUTING -s 192.168.100.0/24 -p tcp --dport 443 -j REDIRECT --to-port 3128
```

 * 443(HTTPS)についてはプロキシを通したからといってキャッシュ効率がよくなるわけではないが、帯域制限の管理下にいれるために設定。
