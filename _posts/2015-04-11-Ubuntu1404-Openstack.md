---
layout: post
title: Ubuntu14.04でOpenstack
---
 * OpenStack本にしたがってやったらあっさりハマる。まぁ本の名誉のためにいっておくと直接は本のせいじゃない（が、本書いた2015/01に本当に通ったのかは少し疑念は残る）
 * P79にあるようにubuntu-virtinst.gitをもってくる（成功）。  
 # git clone https://github.com/josug-book1-materials/ubuntu-virtinst.git  
 * openstack用のネットワークを構築する（成功）。  
 > <network>  
 >   <name>openstack</name>  
 >   <forward mode='nat'/>  
 >   <bridge name='opnestack' stp='off' delay='0'/>  
 >   <ip address='192.168.100.1' netmask='255.255.255.0'>  
 >     <dhcp>  
 >       <range start='192.168.100.200' end='192.168.100.254' />  
 >     </dhcp>  
 >   </ip>  
 > </network>  
 
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
