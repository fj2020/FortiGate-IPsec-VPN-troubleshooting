# FortiGate IPsec VPN troubleshooting

<a href="https://www.fortinet.com/" target="_blank">Fortinet 官网</a>

## IPsec隧道协商失败定位步骤：第一步

先通过sniffer抓包确认用于协商的IKE协议UDP 500是否通信正常，是否中间ISP出现了故障，正常应该有去有回： 


FGT#**diagnose sniffer packet any “host 202.106.2.1 and (port 500 or port 4500)” 4**

        *12.650762 wan1 out 119.100.1.35.500 -> 202.106.2.1.500: udp 716*  

        *12.652838 wan1 in 202.106.2.1.500 -> 119.100.1.35.500: udp 192*  

        *12.664391 wan1 out 119.100.1.35.500 -> 202.106.2.1.500: udp 380*  

        *12.665846 wan1 in 202.106.2.1.500 -> 119.100.1.35.500: udp 380*  



**要注意：有时候需要关闭IPsec第一阶段里的NP加速，才可以抓取到完整的IKE协商过程**


FGT # config vpn ip phase1-interface 

FGT (phase1-interface) # edit Center 

FGT (Center) # **set npu-offload disable**

FGT (Center) # end 


只有确保UDP 500/UDP 4500通信正常，才有接下来协商产生，才有协商成功或失败的定位，因此这是排查的第一步。 


## IPsec隧道协商失败定位步骤：第二步 

然后，通过日志信息以及debug app ike 确认问题是出在Ipsec 协商第一阶段还是第二阶段 

diagnose vpn ike log-filter dst-addr4 *124.65.148.86*           **//把IP换成对方公网IP** 

diagnose debug  application ike  -1 

diagnose debug  enable 


要注意：debug app ike的时候，自己不要主动发起连接，否则可能看不出故障的原因，特别是和友商对接的时候，由于报错的信息格式不一致，友商的报错信息，我们未必可以准确的读出来，而如果是被动接受协商，报错信息则在本地产生，应该一定可以知道协商失败的原因所在，因此我们需要把第一阶段/第二阶段的自动协商关闭。 


**注意一：可能需要关掉一阶段第二阶段的自动协商** 

如果是5.6之后的版本，只需要一条命令就可以完全关闭自己的主动发起的IKE连接请求： 

config vpn ipsec phase1-interface 

    edit VPN-P1（第一阶段名称） 
    
        set passive-mode enable    //永远不主动发起IKE请求，即便使用流量触发，也不主动发起 
        
        next 
        
    end 


如果是旧版本(5.2/5.4)则需要分别关闭第一阶段和第二阶段的自动协商： 

FGT # config vpn ipsec phase1-interface 

FGT (phase1-interface) # edit VPN-P1（第一阶段名称） 

FGT (VPN) # set auto-negotiate disable 

FGT (VPN) # end 


FGT # config vpn ipsec phase2-interface 

FGT (phase1-interface) # edit VPN-P2 （第二阶段名称） 

FGT (VPN) # set auto-negotiate disable 

FGT (VPN) # end 


**注意二：有时候需要重置IPsec VPN的连接** 

（请谨慎使用，所有的VPN都会重新连接IKE，所有的VPN都会中断重连，影响业务，一般不需要使用这个命令） 


diagnose vpn ike gateway flush name to-hub   // 重置某一条VPN的第一阶段连接 

diagnose vpn ike restart      //重新主动发起连接，会中断所有的VPN隧道 

diagnose vpn tunnel reset  //重置第二阶段，会中断所有的VPN隧道 


**重置IPsec VPN通道，有VDOM的情况下：**

FG200D4615810562 # config vdom 

FG200D4615810562 (vdom) # edit root 

FG200D4615810562 (root) # diagnose vpn tunnel reset 

FG200D4615810562 (root) # diagnose vpn ike restart 


**查看IPsec VPN状态命令：** 

diagnose vpn ike gateway list 

diagnose vpn tunnel list 


