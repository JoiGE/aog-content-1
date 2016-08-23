#使用PowerShell在Azure Active Directory中管理域的集成方式

本文适用于已安装Azure AD Connect集成工具，将本地用户与Azure AD集成，并配置了基于密码同步的登录或基于ADFS联合身份验证服务的联合登陆的客户。关于如何安装并配置Azure AD Connect，请参考[将本地标识与 Azure Active Directory 集成](https://www.azure.cn/documentation/articles/active-directory-aadconnect/#install-azure-ad-connect)。

##常见场景

客户可能会因为安全策略需求，需要将已配置了密码同步登录的Azure环境转换为基于ADFS联合身份验证服务的联合登陆。或者因为某些业务需求，需要弃用基于ADFS联合身份验证服务的联合登陆，将Azure的登录方式转换为基于密码同步的登录方式。

##前提条件

- 有效的Azure或Office 365订阅
- Azure AD或Office 365订阅中[已被验证的自定义域名](https://www.azure.cn/documentation/articles/active-directory-add-domain/)
- 本地域内已安装并部署完成的[Azure AD Connect集成工具](https://www.azure.cn/documentation/articles/active-directory-aadconnect/)
- 本地域内已安装并部署完成的ADFS联合身份验证服务
- 本地域的域管理员账号
- 加入本地域，并拥有internet连接的电脑

## 安装PowerShell 模块

Azure AD模块支持安装了默认版本的微软.Net Framework与Windows PowerShell工具的以下操作系统：Windows 8.1，Windows 8，Windows 7，Windows Server 2012 R2，Windows Server 2012或Windows Server 2008 R2。
以域管理员账号登陆电脑，下载并安装[适用于 IT 专业人员 RTW 的 Microsoft Online Services 登录助手](https://www.microsoft.com/zh-cn/download/details.aspx?id=41950)，然后下载并安装[Windows PowerShell Azure AD模块（x64版本）](https://msdn.microsoft.com/zh-cn/library/azure/dn495300.aspx)。

## 连接至Azure AD

以管理员模式运行桌面的Windows Azure Active Directory Module for Windows PowerShell快捷方式，或在普通PowerShell窗口中输入`Import-Module MSOnline`以加载模块指令。
在运行本文介绍的PowerShell指令之前，你必须使用`Connect-MsolService`命令连接至Azure AD，然后输入你的账户与密码（例如：example@domain.partner.onmschina.cn)。你也可以使用以下命令提前输入账户凭据并连接：

	$msolcred = get-credential
	connect-msolservice -credential $msolcred

## 查看Azure AD中域的联合设置

在连接至Azure AD之后，你可以使用以下命令得到当前域名的联合设置信息(将domain.com替换成你的订阅中已被验证的自定义域名)：

	Get-MsolDomain
	Get-MsolDomainFederationSettings -DomainName (domain.com)

## 连接至本地域内的ADFS联合身份验证服务

在连接至Azure AD之后，你需要使用以下命令来使PowerShell能够连接至本地域内的ADFS服务，使得PowerShell能够读取或更改ADFS服务的信息(将adfs.domain.com替换成你的本地域中ADFS服务器的机器名)：

	Set-MsolAdfsContext -Computer (adfs.domain.com)

## 将Azure AD中的域从联合登陆模式转换至密码同步登陆模式

当Azure AD里域的用户使用联合登陆时，他/她使用本地域的用户凭据登陆Azure/Office 365，Azure AD本身不会存有用户的密码哈希。
在域从联合登陆转换模式至密码同步登陆模式时，PowerShell会删除ADFS服务中关于Azure AD的依赖方(Relying Party)设置，并给该域内的每个被同步的用户提供一个临时的密码。以下是转换命令(将domain.com替换成你的订阅中已被验证的自定义域名)：

	Convert-MsolDomainToStandard -DomainName (domain.com) -PasswordFile C:\password.txt -SkipUserConversion $false

临时密码会被储存在C:\password.txt文件中，管理员需要将临时密码分配给每个用户，以便他们登陆Azure/Office 365。
如果管理员希望用户可以沿用之前的密码，可以用Azure AD Connect工具触发一次密码全同步操作，该操作会将被同步用户的密码哈希同步至Azure AD中。关于如何触发密码全同步，请参阅[使用 Azure AD Connect 同步实现密码同步](https://www.azure.cn/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization/)。

## 将Azure AD中的域从密码同步登陆模式转换至联合登陆模式

你可以使用以下命令将Azure AD的域从密码同步登陆模式转换至联合登陆模式(将domain.com替换成你的订阅中已被验证的自定义域名)：

	Convert-MsolDomainToFederated -DomainName (domain.com) -SupportMultipleDomain

PowerShell将在本地域的ADFS服务中注册Azure AD的依赖方(Relying Party)设置，并在Azure AD中更新域的联合登陆设置。

## 更新Azure AD中域的联合登陆配置信息

在某些情况下，你需要更新域的联合登陆配置信息，例如：

- 本地域中ADFS服务的签名证书被更新
- 本地域中ADFS服务中Azure AD的依赖方设置(Relying Party)被误改或误删
- Azure AD的签名证书被更新等

使用以下命令来更新域的联合登陆配置信息(将domain.com替换成你的订阅中已被验证的自定义域名)：

	Update-MsolFederatedDomain -DomainName (domain.com) -SupportMultipleDomain

## 更多信息

- [将自定义域名添加到 Azure Active Directory](https://www.azure.cn/documentation/articles/active-directory-add-domain/)
- [将本地标识与 Azure Active Directory 集成](https://www.azure.cn/documentation/articles/active-directory-aadconnect/#install-azure-ad-connect)

