# SSL/TLS

SSL(Secure Sockets Layer 安全套接字协议),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层与应用层之间对网络连接进行加密。

### SSL会话三步骤

- 客户端向服务端索要并验证证书
- 双方协商生成“会话秘钥“
- 双方采用“会话秘钥”进行加密通信

- 第一阶段：clientHello
- 第二阶段：ServerHello
  - 确认使用的加密通信版本
- 第三阶段：
  - 验证服务器证书（发证机构、证书完整性、有效期、证书持有者）