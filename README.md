# FortiGate IPsec VPN troubleshooting

<a href="https://www.fortinet.com/" target="_blank">Fortinet 官网</a>

## IPsec隧道协商失败定位步骤：第一步 

先通过sniffer抓包确认用于协商的IKE协议UDP 500是否通信正常，是否中间ISP出现了故障，正常应该有去有回： 


FGT#**diagnose sniffer packet any “host 202.106.2.1 and (port 500 or port 4500)” 4**

12.650762 wan1 out 119.100.1.35.500 -> 202.106.2.1.500: udp 716 

12.652838 wan1 in 202.106.2.1.500 -> 119.100.1.35.500: udp 192 

12.664391 wan1 out 119.100.1.35.500 -> 202.106.2.1.500: udp 380 

12.665846 wan1 in 202.106.2.1.500 -> 119.100.1.35.500: udp 380 


**要注意：有时候需要关闭IPsec第一阶段里的NP加速，才可以抓取到完整的IKE协商过程**


FGT # config vpn ip phase1-interface 

FGT (phase1-interface) # edit Center 

FGT (Center) # **set npu-offload disable**

FGT (Center) # end 


只有确保UDP 500/UDP 4500通信正常，才有接下来协商产生，才有协商成功或失败的定位，因此这是排查的第一步。 


## IPsec隧道协商失败定位步骤：第二步 
