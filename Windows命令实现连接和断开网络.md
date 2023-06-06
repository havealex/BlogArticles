# [Windows命令实现连接、断开网络](https://blog.csdn.net/weixin_45906196/article/details/129268942)
## win+r，输入cmd，打开命令行工具
- 断开网络：`netsh wlan disconnect`
- 连接网络：`netsh wlan connect "network_name"`
- 查看网络配置信息,可以查看连接过的网络名称：`netsh wlan show profiles`
- 查看网卡驱动信息：`netsh wlan show drivers`

## 其他常用netsh wlan命令
```
netsh wlan add            - 在一个表格中添加一个配置项。
netsh wlan connect        - 连接到无线网络。
netsh wlan delete         - 从一个表格中删除一个配置项。
netsh wlan disconnect     - 从无线网络断开连接。
netsh wlan dump           - 显示一个配置脚本。
netsh wlan export         - 将 WLAN 配置文件保存为 XML 文件。
netsh wlan help           - 显示命令列表。
netsh wlan IHV            - 用于 IHV 记录的命令。
netsh wlan refresh        - 刷新承载网络设置。
netsh wlan reportissues   - 生成 WLAN 智能跟踪报告。
netsh wlan set            - 设置配置信息。
netsh wlan show           - 显示信息。
netsh wlan start          - 启动承载网络。
netsh wlan stop           - 停止承载网络。
 
netsh wlan show all       - 显示完整的无线设备和网络信息。
netsh wlan show allowexplicitcreds - 显示允许共享用户凭据设置。
netsh wlan show autoconfig - 显示是否启用或禁用自动配置逻辑。
netsh wlan show blockednetworks - 显示阻止的网络显示设置。
netsh wlan show createalluserprofile - 显示是否允许所有人创建所有用户配置文件。
netsh wlan show drivers   - 显示系统上无线 LAN 驱动程序的属性。
netsh wlan show filters   - 显示允许和阻止的网络列表。
netsh wlan show hostednetwork - 显示承载网络的属性和状态。
netsh wlan show interfaces - 显示系统上的无线局域网接口的列表。
netsh wlan show networks  - 显示计算机上可见的网络列表。
netsh wlan show onlyUseGPProfilesforAllowedNetworks - 显示在配置 GP 的网络设置上仅使用 GP 配置文件。
netsh wlan show profiles  - 显示计算机上配置的配置文件列表。
netsh wlan show randomization - 显示 MAC 随机化是已启用还是已禁用。
netsh wlan show settings  - 显示无线 LAN 的全局设置。
netsh wlan show tracing   - 显示是否启用或禁用无线局域网跟踪。
netsh wlan show wirelesscapabilities - 显示系统的无线功能
netsh wlan show wlanreport - 生成一个报告，显示最新的无线会话信息。
```
