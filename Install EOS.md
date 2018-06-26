# 超级详细安装流程

## 安装虚拟机
网上有大把教程，随便找一个照着装就行了

## 安装UBUNTU最新版的系统；
网上有大把教程，随便找一个照着装就行了

## 进入虚拟机并安装VMWARE TOOLS：
    sudo ./vmware-install.pl –d

## 打开终端，安装GIT：
    sudo apt install git

## 下载EOS源代码：
    git clone https://github.com/eosio/eos --recursive
在下载之前一定要设置代理服务器，要不然速度很慢，而且下载时容易漏文件，就会导致后续一大堆的安装问题；

## 安装EOSIO：
    cd eos
    ./eosio_build.sh 

## 安装可执行文件
生成全局命令，写入/usr/local
    cd build
    sudo make install

## 搭建本地EOS运行环境：
    cd eos/build/programs/nodeos
    ./nodeos --config-dir data-dir/ --replay-blockchain 
注：执行了该命令后，会像官方文档所提示的那样报出一个错误，有错误是正确的，没有报错才要慌，这个错误是为了让你修改它的配置文件config.ini。此时查看nodeos，可以发现多了一个名为data-dir的文件夹，打开这个文件下可以看到一个config.ini的配置文件，这时我们用vi编辑器来修改这个文件，这个文件里有很多东西，你可以全部删除，也可以就在里面修改，我的建议是全部删除，方便省事。在这个文件夹下添加如下内容（genesis-json文件的地址一定要写正确）：

```
# Load the testnet genesis state, which creates some initial block producers with the default key
genesis-json = /home/wangwang/eos/tutorials/bios-boot-tutorial/genesis.json
# Enable production on a stale chain, since a single-node test chain is pretty much always stale
enable-stale-production = true
# Enable block production with the testnet producers
producer-name = inita
producer-name = initb
producer-name = initc
producer-name = initd
producer-name = inite
producer-name = initf
producer-name = initg
producer-name = inith
producer-name = initi
producer-name = initj
producer-name = initk
producer-name = initl
producer-name = initm
producer-name = initns
producer-name = inito
producer-name = initp
producer-name = initq
producer-name = initr
producer-name = inits
producer-name = initt
producer-name = initu
producer-name = eosio
# Load the block producer plugin, so you can produce blocks
plugin = eosio::producer_plugin
# Wallet plugin
plugin = eosio::wallet_api_plugin
# As well as API and HTTP plugins
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
# This will be used by the validation step below, to view account history
plugin = eosio::history_api_plugin·
```

保存后再次执行（./nodeos --config-dir data-dir/）
此时屏幕会不停的滚动，系统开始创建区块。做到这里，eos本地环境部署就已经成功了，开始你的EOS之旅吧；

 

# 踩过的坑：
## GITHUB访问慢和CLONE慢解决方案：
可以设置代理通道，设置后下载速度可达到200K/S；
授人以鱼(解决方案)：
ubuntu修改/etc/hosts(windows下C:\Windows\System32\drivers\etc\HOST)文件添加如下ip隐射：
    192.30.253.113 github.com
授人以渔(解决方法)：
通过http://tool.chinaz.com/dns/让到对你来说最快的代理服务器，如：我找到的最快的代理是：192.30.253.113，所以在上魔神的配置文件添加以下配置： 

    192.30.253.113 github.com

## 安装VMWARE TOOLS时可能出现的问题：
输入sudo ./vmware-install.pl –d命令时出现无法安装的问题，那就把“-d”去掉，只输入sudo ./vmware-install.pl命令，然后按提示一步一步来来吧，别偷懒了，“-d”参数是按默认设置安装；

## 安装MONGODB C & C++ DRIVER时可能出现的问题：
可能会出现“make -j”命令格式不正确的错误，这个时间你只去找到/eos/scripts/eosio_build_ubuntu.sh这个文件，把文件内的所有make -j"${JOBS}" 改为 make –j4 最后的数字是你虚拟机CPU内核数量；
这里出现问题的原因，我觉得可能是一键安装的脚本读取不到Ubuntu虚拟机的CPU内核数量导致的；

## 使用./EOSIO_BUILD.SH脚本一键安装的大坑：
在Ubuntu虚拟机安装eos时，此脚本内获取CPU数量的变量会取不到有效值，这个命令就是：make -j"${JOBS}"，双引号内的就是变量，取不到有效值去编译文件时，会把电脑的内存全部吃光，然后虚拟机就死机了；
解决方法：把eosio_build.sh和eosio_build_ubuntu.sh脚本内的make -j"${JOBS}"全部改为make -j$( nproc )

## 访问共享文件夹（VMSHAREDFILES）：
    cd /mnt/hgfs/VMSharedFiles
把目录A下的所有文件拷贝到目录B
    cp -r eos /  /home/wangwang/eos/

## 从WINDOWS拷贝文件到UBUNTUN时需要注意的事：
从windows拷贝eos到Ubuntu时，运行eosio_build.sh会出现找不到文件或者目录的错误，解决方法如下：
原因是windows会自动在文件中添加一些字符，删除就可以了，
    sed -i 's/\r$//' eosio_build.sh
    sed -i 's/\r$//' eosio_build_ubuntu.sh

## NODES 启动报错:
需要把account_history_api_plugin 替换成history_api_plugin
如果以下错误:
```
feng/work/eos/libraries/chainbase/src/chainbase.cpp(106): Throw in function chainbase::database::database(const bfs::path &, chainbase::database::open_flags, uint64_t)
Dynamic exception type: boost::exception_detail::clone_impl<boost::exception_detail::error_info_injector<std::runtime_error> >
std::exception::what: database dirty flag set (likely due to unclean shutdown) replay or resync required
```

### 需要在nodes的时候添加:
    --replay-blockchain
//or
    --resync-blockchain

## EOS源码构建：
注意： 这里笔者按照官方的代码运行出错了，真正有用的代码我写出来，其实就是两个路径的改变
```
cd ~
git clone https://github.com/eosio/eos --recursive
mkdir -p ~/eos/build && cd ~/eos/build
cmake -DBINARYEN_BIN=~/binaryen/bin -DWASM_ROOT=~/wasm-compiler/llvm -DOPENSSL_ROOT_DIR=/usr/include/openssl -DOPENSSL_LIBRARIES=/usr/include/openssl ..
make -j$( nproc )
```

## NEDEOS节点服务无法开启：
删除日志文件：rm -rf ~/.local/share/eosio/nodeos/data
再重新启动节点：nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin

先执行命令：./nodeos --config-dir data-dir/ --delete-all-blocks 删除所有数据；
再执行命令：./nodeos --config-dir data-dir/ --hard-replay-blockchain重新启动节点服务；
正常启动节点服务可使用此简化命令：./nodeos --config-dir data-dir/

## VMWARE 虚拟机的鼠标左右键无法使用的问题：
这是因为虚拟机的声卡和主机的声卡冲突了，把虚拟机的声卡删除就可以了。
这个问题的症状很明显，鼠标左右键点击无效，但是鼠标移动到文件上面会高亮显示，鼠标滚轮也可以用。
如果装了桌面日历软件，那就关掉，它会跟Vmware虚拟机冲突。

## 更新EOS源码：
```
cd eos
git submodule update --init –recursive
```






