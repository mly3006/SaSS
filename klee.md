# klee
## 实验要求
* 安装klee
* 完成完成官方 Tutorials
   * [x] Testing a Small Function
## 实验环境
* kali linux
## 实验环境
* 安装klee
```
apt-get install docker.io
发现安装包依赖损坏,进行如下修复:
sudo apt --fix-broken install
sudo apt-get update
sudo apt-get upgrade
还是不行，应该是linux版本问题，aptitude是可以选择合适的版本与匹配软件安装。
但是安装aptitude也会出现包依赖损坏的问题。。。不知道怎么解决。
