# 游戏网络同步

## 原理文章参考

[林伟的网游同步科普](http://www.skywind.me/blog/archives/1343)

[移动同步的平滑算法](http://www.zhust.com/index.php/2014/02/%E7%BD%91%E7%BB%9C%E6%B8%B8%E6%88%8F%E7%9A%84%E7%A7%BB%E5%8A%A8%E5%90%8C%E6%AD%A5%EF%BC%88%E4%B8%89%EF%BC%89%E5%B9%B3%E6%BB%91%E7%AE%97%E6%B3%95/)

[航位预测法Dead Reckoning](http://www.gamasutra.com/view/feature/131638/dead_reckoning_latency_hiding_for_.php)


## 帝国时代II

2001年作者的技术分享
[在28.8K网络下的1500人的同步](http://www.gamasutra.com/view/feature/131503/1500_archers_on_a_288_network_.php)

帝国时代采用lockstep的变种 ( 动作同步，然后各自重放模拟 )
执行step比通信step延迟2个turn:
- local:  发出当前commands,  2个turn后再执行
- remote: 收到command turn为1000, 则在本地的turn:1002时执行

### Speed Control:  
	通过计算网络延迟及计算能力，修改communication turn的长度
	仲裁机收集了网络延迟和渲染帧率，然后广播调整每个turn的时间
### 网络层:
	 本turn收不到包，发起重传请求，让对方重传。(NACK机制)

## 守望先锋

[Let's Talk Netcode|Overwatch](https://www.youtube.com/watch?v=vTH2ZPgYujQ&feature=youtu.be)

upload 60Hz, download 20Hz
大量的客户端prediction
     按下前进，本地会做simulation，同时发指令给server

Adaptive intepolation

对于高ping玩家，采用外推(extrapolation). 
server会buffer过去的4个指令，然后反推。但是反推太过的话，网速好的玩家就会感觉被干。所以这里是个tradeoff

## Valve的source引擎 

dota2, CS online的主要引擎
[引擎Networking部分](https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking)
[运动补偿](https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization#Footnotes)

### Input prediction
	cl_predict   客户端预测  (input之后不等服务端，本地先行)
	如果下个snapshot发现本地与服务端不一致(prediction_error) ,是否能通过smoothtime来平滑，如果不能，直接跳。
### 预测补偿
	Command Execution Time = Current Server Time - Packet Latency - Client View Interpolation

### Entity Intepolation
	Then the interpolation could use snapshots 340 and 344. If more than one snapshot in a row is dropped, interpolation can't work perfectly because it runs out of snapshots in the history buffer. In that case the renderer uses extrapolation (cl_extrapolate 1) and tries a simple linear extrapolation of entities based on their known history so far. The extrapolation is done only for 0.25 seconds of packet loss (cl_extrapolate_amount), since the prediction errors would become too big after that.

要点：source引擎抽象出几个参数：先行预测，根据rtt预测补偿，client对snapshot插值(包括内插和外插)


## Dota2

[关于dota2的网络讨论](http://dev.dota2.com/showthread.php?t=527&page=7&p=4253&viewfull=1#post4253)

因为源于source引擎，跟source是一样的，S->C发snapshot, 插值。dota2没有做client prediction

## Glenn Fiedler博客

[物理模拟的网络同步专题](http://gafferongames.com/game-physics/)