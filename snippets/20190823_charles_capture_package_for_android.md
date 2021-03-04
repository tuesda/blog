### Charles 在 Android 设备上抓包
- 电脑上关掉 surge 等代理翻墙软件
- 手机上 Wi-Fi 设置代理 IP 和 Port
    - IP：电脑局域网 IP 地址，查看方式：打开 Charles  `Help - Local IP Address`
    - Port：打开 Charles `Proxy - Proxy Settings... - Proxies` 查看或设置

这时电脑上的 Charles 就可以抓 Android 设备上的包了。

但现在大部分请求为了安全都用的是 `https://` 加密连接，Charles 上看到的都是乱码，为了看到解密后的明文，需要做这几件事：

- 打开手机浏览器在地址栏输入 `http://www.charlesproxy.com/getssl` 安装证书

- Android 7.0 后的设备需要一些额外配置，用来允许上一步安装的证书

    - 在 `main/res/xml/` 新建 `network_security_config.xml` 配置文件：

        ``` xml
        <network-security-config>
            <debug-overrides> // debug
                <trust-anchors>
                    <certificates src="user" />
                    <certificates src="system" />
                </trust-anchors>
            </debug-overrides>
            <base-config> // 全局配置
                <trust-anchors>
                    <certificates src="user" /> // 主要是这一行，表示信任自己安装的证书
                    <certificates src="system" />
                </trust-anchors>
            </base-config>
        </network-security-config>
        ```

    - 将上面创建的配置文件设置在 `AndroidManifest.xml` 中

        ``` xml
        ...
        <application android:networkSecurityConfig="@xml/network_security_config">
        ...
        ```

- Charles 上打开 `Proxy - SSL Proxying Settings... - SSL Proxying`，加上需要解密的域名：

    ```
    *.host.com
    ```



> References:
>
> - https://medium.com/@daptronic/the-android-emulator-and-charles-proxy-a-love-story-595c23484e02



