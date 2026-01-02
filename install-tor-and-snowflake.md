在 macOS 上安装并使用 Tor 和 Snowflake 实现国际互联网访问

本文档介绍如何在 macOS 上安装和配置 Tor 以及 Snowflake 客户端，以便能够访问国际互联网服务。Tor 是一个免费的开源软件，旨在实现匿名通信，而 Snowflake （参考链接：https://support.torproject.org/zh-CN/anti-censorship/what-is-snowflake/） 是 Tor 网络中的一种桥接器，利用 WebRTC （webrtc.org）技术帮助用户连接到 Tor 网络从而绕过网络封锁。


## 最简单的安装方法（推荐初学者使用）
1. 下载并安装 Tor Browser（参考链接：https://www.torproject.org/download/）
2. 打开 Tor Browser，打开“设置”页面。
3. 在“设置”页面中，左侧列表中找到“连接”并点击。
4. 右侧面板中找到“桥接”，勾选“使用桥接器”，然后点击下面的“选择一个内置桥接器”。
5. 勾选“Snowflake”并“确定”关闭窗口。
6. 点击“连接”按钮，Tor Browser 将尝试通过 Snowflake 桥接器连接到 Tor 网络。
7. 连接成功后，您可以通过 Tor Browser 访问国际互联网服务。
8. 如果需要在本机其他应用程序中使用 Tor 代理，可以参考下文的“设置”部分进行相应的配置。代理地址是 `socks5h://localhost:9050`。

## 如果需要局域网其它设备也可以使用 Tor 代理，请继续阅读下面的内容进行安装和配置。

## 安装 Snowflake 和 Tor 的先决条件
1. 确保您的 macOS 系统已更新到最新版本。
2. 安装 Homebrew（如果尚未安装）：
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

## 使用 brew 安装 Snowflake（参考链接：https://snowflake.torproject.org/）
1. 打开终端应用程序。
2. 使用 Homebrew 安装 Snowflake：
   ```bash
   brew install snowflake
   ```

## 使用 brew 安装 Tor （参考链接：https://support.torproject.org/zh-CN/little-t-tor/getting-started/installing/#macos）
1. 打开终端应用程序。
2. 使用 Homebrew 安装 Tor：
   ```bash
   brew install tor
   ```
3. 安装完成后，编辑 Tor 的配置文件 `torrc`：
   ```bash
   cp /usr/local/etc/tor/torrc.example /usr/local/etc/tor/torrc
   nano /usr/local/etc/tor/torrc
   ```
4. 在 `torrc` 文件中，添加以下配置以启用 SOCKS 和 HTTP 隧道：
- 仅供本机使用：
   ````
   SOCKSPort localhost:9050
   HTTPTunnelPort localhost:9150  # 主要供苹果手机使用
   ``````
- 供局域网使用（根据需要修改 <your-macos-computername>）：
   ````
   SOCKSPort <<your-macos-computername>>.local:9050
   HTTPTunnelPort <<your-macos-computername>>.local:9150  # 主要供苹果手机使用
   ``````
5. 配置 Snowflake 桥接器（参考链接：https://gitlab.torproject.org/tpo/anti-censorship/pluggable-transports/snowflake/-/tree/main/client?ref_type=heads）：
   ```
   UseBridges 1
   ClientTransportPlugin snowflake exec /usr/local/bin/snowflake-client
   Bridge snowflake 192.0.2.4:80 8838024498816A039FCBBAB14E6F40A0843051FA fingerprint=8838024498816A039FCBBAB14E6F40A0843051FA url=https://1098762253.rsc.cdn77.org/ fronts=www.cdn77.com,www.phpmyadmin.net ice=stun:stun.antisip.com:3478,stun:stun.epygi.com:3478,stun:stun.uls.co.za:3478,stun:stun.voipgate.com:3478,stun:stun.mixvoip.com:3478,stun:stun.nextcloud.com:3478,stun:stun.bethesda.net:3478,stun:stun.nextcloud.com:443 utls-imitate=hellorandomizedalpn
   Bridge snowflake 192.0.2.3:80 2B280B23E1107BB62ABFC40DDCC8824814F80A72 fingerprint=2B280B23E1107BB62ABFC40DDCC8824814F80A72 url=https://1098762253.rsc.cdn77.org/ fronts=www.cdn77.com,www.phpmyadmin.net ice=stun:stun.antisip.com:3478,stun:stun.epygi.com:3478,stun:stun.uls.co.za:3478,stun:stun.voipgate.com:3478,stun:stun.mixvoip.com:3478,stun:stun.nextcloud.com:3478,stun:stun.bethesda.net:3478,stun:stun.nextcloud.com:443 utls-imitate=hellorandomizedalpn
   ```
6. 启用 DNS 泄漏保护（可选但强烈推荐）：
   ```
   SafeSocks 1
   ```
7. 保存并关闭文件。
8. 启动 Tor 服务：
   ```bash
   brew services start tor
   ```
9. Tor 初次连接时可能需要一些时间，请耐心等待。

## 设置终端中的环境变量以使用 Tor 代理（全局设置，会影响所有使用了环境变量的程序）
1. 打开终端应用程序。
2. 编辑您的 shell 配置文件（例如 `.zshrc` 或 `.bash_profile`）：
   ```bash
   nano ~/.zshrc  # 如果使用 zsh
   ```
   或
   ```bash
   nano ~/.bash_profile  # 如果使用 bash
   ```
3. 添加以下行以设置环境变量，具体使用 socks5h:// 还是 http:// 协议依具体程序，常见程序一般都支持这两种协议：
   ```bash
   export ALL_PROXY="socks5h://localhost:9050"
   export HTTP_PROXY="socks5h://localhost:9150"
   export HTTPS_PROXY="socks5h://localhost:9150"
   ```
## 设置个别程序使用 Tor 代理，不影响全局设置和其它程序
   ```bash
   HTTP_PROXY="socks5h://localhost:9150" HTTPS_PROXY="socks5h://localhost:9150" git clone <repository-url>
   ```
## 设置浏览器使用 Tor 代理
1. 打开您常用的浏览器（如 Firefox 或 Chrome）。
2. 配置浏览器的代理设置：
   - SOCKS 代理地址：`localhost` 或 `<your-macos-computername>.local`，端口 `9050`
   - HTTP 代理地址：`localhost` 或 `<your-macos-computername>.local`，端口 `9150`
3. 保存设置并访问 [https://check.torproject.org](https://check.torproject.org) 以验证连接是否通过 Tor。
## 设置 Git 使用 Tor 代理
1. 在终端中运行以下命令以配置 Git 使用 Tor socks 代理（如果担心 DNS 泄漏，请务必使用 socks5h:// 协议：
- 本机使用：
   ```bash
   git config --global http.proxy 'socks5h://localhost:9050'
   git config --global https.proxy 'socks5h://localhost:9050'
   ```
- 局域网使用：
   ```bash
   git config --global http.proxy 'socks5h://<your-macos-computername>.local:9050'
   git config --global https.proxy 'socks5h://<your-macos-computername>.local:9050'
   ```
2. 在终端中运行以下命令配置 Git 使用 Tor http 代理（http:// 协议默认使用代理解析 DNS，不存在泄漏的问题）：
- 本机使用：
   ```bash
   git config --global http.proxy 'http://localhost:9150'
   git config --global https.proxy 'http://localhost:9150'
   ```
- 局域网使用：
   ```bash
   git config --global http.proxy 'http://<your-macos-computername>.local:9150'
   git config --global https.proxy 'http://<your-macos-computername>.local:9150'
   ```
3. 验证 Git 代理设置：
   ```bash
   git config --global --get http.proxy
   git config --global --get https.proxy
   ```
## 设置苹果手机使用 Tor 代理（只能使用 HTTP 代理）
1. 在 iPhone 上，进入“设置” > “Wi-Fi”。
2. 点击您当前连接的 Wi-Fi 网络旁边的“信息”图标。
3. 向下滚动到“HTTP 代理”部分，选择“手动”。
4. 在“服务器”字段中输入 `<your-macos-computername>.local`，在“端口”字段中输入 `9150`。
5. 保存设置。
6. 打开 Safari 或其他浏览器，访问 [https://check.torproject.org](https://check.torproject.org) 以验证连接是否通过 Tor。


## 验证连接
1. 打开浏览器，访问 [https://check.torproject.org](https://check.torproject.org) 以验证您是否通过 Tor 连接。
2. 如果显示“您正在使用 Tor”，则说明配置成功。

## 其他配置选项
1. 修改日志级别，方便定位问题：
   - 如果需要更改 Tor 的日志级别，可以在 `torrc` 文件中找到：
     ```
     #Log debug file /usr/local/var/log/tor/debug.log
     ```
   - 改为：
     ```
     Log debug file /usr/local/var/log/tor-debug.log
     ```
     修改日志文件路径和名称为了避免因为权限问题导致日志无法写入。默认情况下 /usr/local/var/log/tor/ 是不存在的。
   - 查看日志，以便定位问题：
        ```bash
        tail -f /usr/local/var/log/tor.log
        ```
        ```bash
        tail -f /usr/local/var/log/tor-debug.log
        ```
- 可以根据需要调整 SOCKS 和 HTTP 代理的端口号，以避免端口冲突。
- 确保防火墙允许所使用的端口进行通信。当局域网用户首次使用 <your-macos-computername>.local 或者 <your-macos-ip> 连接时，macOS 会询问是否接受外部连接，此时需要点击“接受”，Tor 代理才能被局域网用户连接使用。

## 停止 Tor 服务
- 如果需要停止 Tor 服务，可以使用以下命令：
  ```bash
  brew services stop tor
  ```
## 重启 Tor 服务
- 如果需要重启 Tor 服务，可以使用以下命令：
  ```bash
  brew services restart tor
  ```

## 常见问题
- **定期更新**：建议定期更新 Tor 和 Snowflake 以获取最新的安全补丁和功能改进。可以使用以下命令更新：
  ```bash
  brew update
  brew upgrade tor snowflake
  ```
- **mDNS 和 .local 域名**：macOS 使用 mDNS（Multicast DNS）解析 `.local` 域名，这允许在本地网络中通过计算机名访问设备，而无需配置静态 IP 地址。确保您的计算机名正确设置，并且在局域网内其他设备上也能解析该名称。
- **查看和编辑本地计算机名**：
  * 可以通过以下命令查看您的 macOS 计算机名：
    ```bash
    scutil --get LocalComputerName # 查询本地计算机名
    scutil --set LocalComputerName <your-macos-computername> # 设置新的本地计算机名
    ```
  * 或者：
    1. 在 macOS 上，打开“设置”。
    2. 在 “搜索” 框中输入 "hostname"。
    3. 在搜索结果中找到并点击“共享”。
    4. 在“计算机名”字段中查看您的本地计算机名。
    5. 如果需要更改计算机名，可以点击“编辑”按钮进行修改。
    6. 修改完成后，点击“好”保存更改。
  * 这样就可以在其它局域网设备上使用 `<your-macos-computername>.local` 来访问您的 macOS 而不用担心 IP 地址变动导致无法访问的情况出现了。
- ** 检查 <your-macos-computername>.local 解析是否正常**：
  - 在终端中使用以下命令检查 mDNS 解析是否正常工作：
    ```bash
    ping <your-macos-computername>.local
    ```
  - 如果执行的结果是本机的局域网地址，说明解析正常；如果无法解析，或解析为 127.0.0.1，则说明 mDNS 工作异常。
  - 首先检查 /etc/hosts 文件，如果其中有 `<your-macos-computername>.local` 条目，请务必删除。
  - 如果 /etc/hosts 文件没有问题，但仍然无法解析，可以尝试重启 mDNS 服务：
    ```bash
    sudo killall -HUP mDNSResponder
    ```
  - 稍等片刻后，再次尝试 ping 命令，看看是否能够正确解析。
- **Socks5 和 Socks5h 的区别**：Socks5h 会通过代理服务器进行 DNS 解析，而 Socks5 则不会。如果您希望通过 Tor 进行 DNS 解析，请使用 Socks5h （参考链接：https://stackoverflow.com/questions/74813567/how-to-configure-gits-sock5-proxy-to-use-remote-dns-resolve）。
- **应用程序 DNS 泄漏**：主要是指应用程序通过自身逻辑或其它外部程序完成 DNS 解析，只有连接目标服务时通过 Tor 完成，这样用户访问目标服务时的全部流程没有在 Tor 网络内进行，存在信息泄漏的问题。（参考链接：https://support.torproject.org/zh-CN/little-t-tor/troubleshooting/check-for-leaks/）
- **Git 查看详细日志**：可以通过设置环境变量 `GIT_CURL_VERBOSE=1` 和 `GIT_CURL_VERBOSE=1` 来查看 Git 的详细日志信息，帮助排查问题。例如：
  ```bash
  GIT_CURL_VERBOSE=1 GIT_VERBOSE=1 git clone <repository-url>
  ```
- **brew 查看包安装位置**：可以使用 `brew --prefix <package-name>` 来查看 Homebrew 包的安装位置。例如：
  ```bash
  brew --prefix tor
  ```
  这将显示 Tor 包的安装路径，方便您查找相关文件。
- **本文档介绍的方法没有做分流**：即所有流量都会通过 Tor 网络进行传输。如果您希望仅对特定流量进行代理，可以考虑配合其他工具（如 shadowrocket：https://apps.apple.com/us/app/shadowrocket/id932747118
Shadowrocket）来实现分流功能。

## 参考链接
- [Snowflake 官方网站 - 中文](https://snowflake.torproject.org/zh-CN/)
- [Snowflake 源码仓库](https://gitlab.torproject.org/tpo/anti-censorship/snowflake)
- [Tor 官方安装指南（macOS）- 中文](https://support.torproject.org/zh-CN/little-t-tor/getting-started/installing/#macos)
- [Tor 项目常见问题解答 - 中文](https://support.torproject.org/zh-CN/little-t-tor/troubleshooting/faq/)
- [Tor 技术支持 - 中文](https://support.torproject.org/zh-CN/)
- [Tor 项目 GitLab](https://gitlab.torproject.org/)
- [Tor 源码仓库](https://gitlab.torproject.org/tpo/core/tor)

## 鸣谢
感谢 Tor 项目团队和社区成员为实现自由开放的互联网所做的贡献。