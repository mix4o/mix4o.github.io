---
layout: post
title: おうちNAS2015構築-省電力&SSD対応
---
 * 省電力
   - 当初cpufreqdいれてondemandにすればいいと考えていたのだが、cpufreq-set -g ondemandとかするとエラーになる。
   - どうもintel_pstateドライバではperformanceとpowersaveにしか対応していないようだ。それは誤算。
   - 仕方ないのでpowersaveにしてみたのだが、クロックは2.3GHz程度までしか下がらないし、どうも期待していたものと違う。もちろんcpufreqd.confはいじってmaxもminも20%に指定している。
   - 次にthermaldをいれてみたのだが、これも期待したものと違う。というか、たぶんこれCPU温度みて高ければ抑えるもので、不必要なら省電力にするというものではないような気がする。
   - Ubuntu固有のアプローチはあまりとりたくなかったが、無駄に電力を食うのは嫌なので[Ubuntu搭載ノートのやっておくべき設定 ～Ubuntu 13.10 省電力関連設定～](http://orebibou.com/2014/03/ubuntu%E6%90%AD%E8%BC%89%E3%83%8E%E3%83%BC%E3%83%88%E3%81%AE%E3%82%84%E3%81%A3%E3%81%A6%E3%81%8A%E3%81%8F%E3%81%B9%E3%81%8D%E8%A8%AD%E5%AE%9A-%EF%BD%9Eubuntu-13-10-%E7%9C%81%E9%9B%BB%E5%8A%9B%E9%96%A2/)を参考に以下。  
     　sudo add-apt-repository ppa:linrunner/tlp  
     　sudo apt-get update  
　     sudo apt-get install tlp tlp-rdw  
   - そして結果はthermaldの時と変わらずクロック数は高いまま。powertopでみるとC6-HSWとかは高いが、これもthermaldの時と変わらないし。i7zでみるとC1が主体でC6になんかほとんど落ちてない。省電力機構が有効に機能しているのかどうかこれまたわからないな。

* SSD
   - SSDの寿命を無駄に短くしないように[Ubuntu LinuxでSSDの寿命を延ばすための設定](http://oinume.hatenablog.com/entry/wp/299)を参考にして設定。
