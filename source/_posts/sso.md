---
title: 「家里云」SSO系统的搭建
date: 2026-07-10
tags: 家里云
---

# 前言
这两天更新服务器的时候注意到之前当作RDP客户端的Next-Terminal还在跑, 决定再利用一下，把自己几个分散的服务器资产都添加进来方便管理。 偶然注意到Next-Terminal是可以做OIDC Server使用的，于是便把一些Web服务也接入了进来。
## SSO与OIDC
SSO（Single Sing-On， 即单点登录）是一种身份验证机制，它允许用户只需输入一次凭据（如用户名和密码），就能访问多个相互信任但独立的软件系统或应用，极过于繁琐的多次登录流程。常见的微信登录、GitHub登录等都属于SSO应用。 而OIDC则是一种建立在 OAuth 2.0 框架之上的轻量级身份层协议，它通过引入“ID Token（身份令牌）”的方式，专门用于在多应用环境下安全地验证用户身份，是当今实现现代 SSO 架构最主流、最核心的技术标准之一。

# OIDC Server
Next-Terminal中只需 *系统设置*-*身份提供服务* 开启OIDC Server、设置Issuer URL。
随后在*身份认证*-*OIDC客户端*中添加应用即可。
![启用OIDC Server](/images/1225.png)
![添加OIDC Client](/images/1226.png)

# OIDC Client
## Synologo DSM
<font color=gray>以DSM 7.2为例</font>
在*域/LDAP*-*SSO客户端*-*OpenID Connect SSO设置*中进行配置，重定向URI设置为nas的访问地址，以“/”结尾。原样照搬到Next-Terminal的应用中。
![](/images/1227.png)
![](/images/6144.jpg)

## Proxmox VE
pve的sso设置藏在*Data Center-Realms*里面，需要新添加一个*OpenID Connect Server*类别的领域，所以既有的用户并不能使用sso登录，sso领域的用户也无法使用密码进行登录。由于我平时用的是root~~（这不是好习惯，但在自己内网也无伤大雅）~~，只得又创建了一个用户赋予PVEAdmin权限……
pve有一点不同的是在登录过程中的回调地址是根据登录时的访问地址确定的，不需要人为设置。可以在Next-Terminal的回调地址中添加所有可能用到的访问地址（例如IP、域名），并且不需要以“/”结尾。
![](/images/6146.jpg)

# 踩坑
- ***<font color=red>在低版本的Next-Terminal中OIDC Server存在签名缺少kid的问题，升级到v3.1.1即可解决。 折腾了我半天……</font>***
- ***<font color=red>DSM如果登录时请求多个范围（openid profile）会卡在授权确认页，目前在Next-Terminal启用了免认证解决问题~~（依旧自己用无伤大雅）~~。不确定更高版本的dsm有没有解决这个问题。</font>***

# 总结
OIDC Server是Next-Terminal在2025年11月底才上的新功能，用起来各种小问题还是有一些，不过当作堡垒机的附带功能还是可以用的。如果是严肃的场景诸如Zitadel一类比较成熟的IdP应该是一个更成熟的选择，通过IdP授权Next-Terminal和其他Web资产，而非把Next-Terminal当作IdP来使用。但话说回来，对于我这样娱乐性的应用直接用Next-Terminal的成本自然低过另外部署一个Zitadel。