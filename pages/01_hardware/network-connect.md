\n\n    
    
    网络是怎样连接的？ - 架构师学习笔记
    
    
    \n\n    
        
            
                网络是怎样连接的？
            
            How Networks Connect - 深入理解网络连接的原理
            
            
                网络连接总览
                
                    
                        
                            1. 应用层
                            Application
                        
                        →
                        
                            2. 传输层
                            Transport
                        
                        →
                        
                            3. 网络层
                            Network
                        
                        →
                        
                            4. 数据链路层
                            Data Link
                        
                    
                
            
            
            
                OSI七层模型
                
                    
                        
                            应用层
                            Application
                        
                        ↓
                        
                            表示层
                            Presentation
                        
                        ↓
                        
                            会话层
                            Session
                        
                        ↓
                        
                            传输层
                            Transport
                        
                        ↓
                        
                            网络层
                            Network
                        
                        ↓
                        
                            数据链路层
                            Data Link
                        
                        ↓
                        
                            物理层
                            Physical
                        
                    
                
            
            
            
                TCP/IP四层模型
                
                    
                        
                            
                            应用层
                        
                        提供应用程序之间的通信。
                        
                            HTTP/HTTPS
                            FTP
                            SMTP
                            DNS
                            SSH
                        
                    
                    
                    
                        
                            
                            传输层
                        
                        提供端到端的通信。
                        
                            TCP (Transmission Control Protocol)
                            UDP (User Datagram Protocol)
                            SCTP (Stream Control Transmission Protocol)
                        
                    
                    
                    
                        
                            
                            网络层
                        
                        提供网络间的路由。
                        
                            IP (Internet Protocol)
                            ICMP (Internet Control Message Protocol)
                            ARP (Address Resolution Protocol)
                        
                    
                    
                    
                        
                            
                            数据链路层
                        
                        提供物理网络的访问。
                        
                            Ethernet
                            Wi-Fi
                            PPP (Point-to-Point Protocol)
                            ATM (Asynchronous Transfer Mode)
                        
                    
                
            
            
            
                网络数据传输过程
                
                    数据发送过程
                    当你在浏览器中输入网址并按下回车时：
                    
                        应用层：浏览器生成HTTP请求
                        传输层：HTTP请求被封装成TCP段
                        网络层：TCP段被封装成IP数据包
                        数据链路层：IP数据包被封装成以太网帧
                        物理层：以太网帧通过物理介质传输
                    
                
                
                
                    数据接收过程
                    服务器接收到数据后：
                    
                        物理层：接收以太网帧
                        数据链路层：解封装以太网帧，得到IP数据包
                        网络层：解封装IP数据包，得到TCP段
                        传输层：解封装TCP段，得到HTTP请求
                        应用层：服务器处理HTTP请求，生成响应
                    
                
            
            
            
                网络寻址
                
                    
                        
                            
                            IP地址
                        
                        用于在网络中唯一标识设备。
                        
                            IPv4：32位地址，如 192.168.1.1
                            IPv6：128位地址，如 2001:0db8:85a3:0000:0000:8a2e:0370:7334
                            公网IP：全球唯一
                            私网IP：局域网内使用
                        
                    
                    
                    
                        
                            
                            MAC地址
                        
                        网络接口的物理地址。
                        
                            48位地址，如 00:1B:44:11:3A:B7
                            由网卡厂商分配
                            在数据链路层使用
                            用于局域网内通信
                        
                    
                
            
            
            
                网络路由
                
                    路由过程
                    数据从源到目的地的路由过程：
                    
                        源主机确定目标IP地址
                        检查目标是否在同一局域网内
                        如果是，直接发送到目标
                        如果不是，发送到默认网关
                        路由器根据路由表转发数据包
                        数据包经过多个路由器到达目的地
                    
                
            
            
            
                网络协议
                
                    
                        
                            
                            HTTP/HTTPS
                        
                        用于网页浏览的协议。
                        
                            HTTP：明文传输
                            HTTPS：加密传输
                            基于TCP协议
                            端口：80/443
                        
                    
                    
                    
                        
                            
                            SMTP/POP3/IMAP
                        
                        用于电子邮件的协议。
                        
                            SMTP：发送邮件
                            POP3：接收邮件
                            IMAP：接收邮件（更高级）
                        
                    
                    
                    
                        
                            
                            DNS
                        
                        域名解析协议。
                        
                            将域名转换为IP地址
                            基于UDP协议
                            端口：53
                            分布式系统
                        
                    
                    
                    
                        
                            
                            SSH
                        
                        安全远程登录协议。
                        
                            加密传输
                            基于TCP协议
                            端口：22
                            替代Telnet
                        
                    
                
            
            
            
                网络安全
                
                    网络安全的重要考虑因素：
                    
                        防火墙：控制网络访问
                        加密：保护数据传输
                        认证：验证用户身份
                        入侵检测：检测异常行为
                        安全更新：修复漏洞
                    
                
            
            
            
                总结
                
                    网络连接是一个复杂但有序的过程，通过分层架构和各种协议，实现了设备之间的数据传输。
                    理解网络连接的原理，有助于我们更好地设计和维护网络系统，也为学习更高级的网络知识打下基础。
