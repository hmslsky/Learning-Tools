# 日常配置
## vim 输入法自动切换
### Window Config
```yaml
[remote-config]
Vim › Auto Switch Input Method: Enable: true
Vim › Auto Switch Input Method: Default IM: 1033
Vim › Auto Switch Input Method: Obtain IMCmd: D:\Applications\BIN\im-select.exe
Vim › Auto Switch Input Method: Switch IMCmd: D:\Applications\BIN\im-select.exe {im}
```

### Linux Config
```yaml
[remote-config]
Vim › Auto Switch Input Method: Enable: true
Vim › Auto Switch Input Method: Default IM: 1
Vim › Auto Switch Input Method: Obtain IMCmd: /usr/bin/fcitx-remote
Vim › Auto Switch Input Method: Switch IMCmd: /usr/bin/fcitx-remote -t {im}
```

### Mac Config

```json
[remote-config]
"vim.autoSwitchInputMethod.enable": true,
"vim.autoSwitchInputMethod.defaultIM": "com.apple.keylayout.ABC", // 这是你的系统英文输入法，我这里用的是 ABC
"vim.autoSwitchInputMethod.obtainIMCmd": "「这里是 which im-select 得到的路径」",
"vim.autoSwitchInputMethod.switchIMCmd": "「这里是 which im-select 得到的路径」 {im}", // 注意 {im} 前面有空格
```

---

## VSCode 使用本地 VPN 代理配置

在 VSCode（Visual Studio Code）中使用本地 VPN 代理，主要有两种常见方式：一是通过 VSCode 的设置文件（settings.json）直接配置代理，二是通过操作系统的环境变量来实现。

### 方法一：通过 VSCode 设置文件配置代理

1. 打开 VSCode。
2. 点击左下角齿轮图标，选择"设置"。
3. 在右上角搜索框输入"proxy"，找到"编辑 in settings.json"。
4. 在 settings.json 文件中添加如下内容（请将 your-proxy-server 和 port 替换为你本地 VPN 的代理地址和端口）：

```json
"http.proxy": "http://your-proxy-server:port",
"https.proxy": "http://your-proxy-server:port",
"http.proxyStrictSSL": false
```

- 如果你的 VPN 代理是 SOCKS5，可以写成：
```json
"http.proxy": "socks5://127.0.0.1:1080"
```
- 保存后，重启 VSCode 即可。

### 方法二：通过环境变量配置代理

#### macOS/Linux 下

1. 打开终端，输入以下命令（同样替换为你的代理地址和端口）：

```bash
export HTTP_PROXY="http://your-proxy-server:port"
export HTTPS_PROXY="http://your-proxy-server:port"
```

- 如果是 SOCKS5 代理：
```bash
export ALL_PROXY="socks5://127.0.0.1:1080"
```

2. 这些命令只对当前终端会话有效。如果希望永久生效，可以将上述命令添加到 `~/.bashrc`、`~/.zshrc` 或 `~/.bash_profile` 文件中，然后执行 `source ~/.zshrc` 使其生效。

3. 重新启动 VSCode（建议用设置好代理的终端执行 `code .` 启动 VSCode）。

### 注意事项

- 代理地址和端口一定要和你的本地 VPN 设置一致。
- 如果代理需要认证，可以在 URL 中加入用户名和密码，例如：
  ```
  http://username:password@your-proxy-server:port
  ```
- 某些插件或扩展可能不完全遵循 VSCode 的代理设置，遇到问题可查阅插件文档。
- 如果遇到 SSL 证书错误，可以将 `"http.proxyStrictSSL": false` 加入 settings.json 以跳过严格验证。

### 参考资料

- [怎么让vscode走代理？](https://blog.csdn.net/m0_57236802/article/details/132169445)