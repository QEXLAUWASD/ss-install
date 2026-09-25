# Auto Install Server Shell Script

- Intro: Auto Install Proxy Server
- System Requirement: CentOS 6+，Debian 7+，Ubuntu 12+

## How to install server:
``` bash
wget --no-check-certificate https://raw.githubusercontent.com/RedTeaDev/ss-install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
**How to uninstall server**
``` bash
./shadowsocks.sh uninstall
```

## Shadowsocks-Rust 多端口與效能設定

安裝時選擇 `Shadowsocks-Rust`，輸入主要端口後可用逗號加入其他端口，例如 `8389,8390`。安裝器會在 `/etc/shadowsocks-rust/config.json` 的 `servers` 清單建立各端口，使用同一密碼和加密方法，並由一個 `ssserver` 程序同時服務。CentOS 的安裝流程會開放這些端口；其他系統及雲端防火牆請自行開放 TCP/UDP。每個客戶端仍須指定端口；若要跨端口分流，請在客戶端或外部負載平衡器設定。

需要獨立程序時，systemd 模板 `ssserver@.service` 會讀取 `/etc/shadowsocks-rust/<名稱>.json`。例如先建立 `edge.json`（使用不同於主配置的端口），以 `chgrp "$(id -gn nobody)" /etc/shadowsocks-rust/edge.json && chmod 640 /etc/shadowsocks-rust/edge.json` 授予服務讀取權限，再執行 `systemctl enable --now ssserver@edge`。請勿讓兩個實例綁定相同端口。可用 `systemctl status ssserver@edge` 查看狀態；移除前以 `systemctl disable --now ssserver@edge` 停止額外實例。

安裝器另寫入 `/etc/sysctl.d/90-shadowsocks-rust.conf` 並套用適度的連線佇列與本機端口範圍設定。若內核提供 BBR，會加上 `net.ipv4.tcp_congestion_control=bbr` 與 `net.core.default_qdisc=fq`；可用 `sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc` 檢查。systemd 服務的 `LimitNOFILE` 為 1048576；SysV 啟動腳本嘗試設為 65535。實際上限仍受主機與容器限制影響。解除安裝會移除安裝器建立的 sysctl 檔案與 systemd 模板；已建立的額外服務須先自行停止，sysctl 執行中的值在重新開機前可能仍然保留。
**How to upgrade server (only support shadowsocks-libev now)**
```bash
./shadowsocks.sh upgrade
```
****

**How to start | stop | restart your server**

Shadowsocks-libev：
/etc/init.d/shadowsocks-libev start | stop | restart | status

ShadowsocksR：
/etc/init.d/shadowsocks-r start | stop | restart | status

****
**Configuration Files**

Shadowsocks-libev ：
/etc/shadowsocks-libev/config.json

ShadowsocksR ：
/etc/shadowsocks-r/config.json

****

**Ciphers（Shadowsocks-libev）:**
aes-256-gcm
aes-192-gcm
aes-128-gcm
aes-256-cfb
aes-192-cfb
aes-128-cfb
aes-256-ctr
aes-192-ctr
aes-128-ctr
camellia-256-cfb
camellia-192-cfb
camellia-128-cfb
xchacha20-ietf-poly1305
chacha20-ietf-poly1305 (Prefered)
chacha20-ietf
chacha20
salsa20
bf-cfb
rc4-md5

**Ciphers（none means unencrypted，ShadowsocksR）:**
none
aes-256-cfb
aes-192-cfb
aes-128-cfb
aes-256-cfb8
aes-192-cfb8
aes-128-cfb8
aes-256-ctr
aes-192-ctr
aes-128-ctr
chacha20-ietf
xchacha20
xsalsa20
chacha20
salsa20
rc4-md5

**Protocols（Only ShadowsocksR）:**
origin
verify_deflate
auth_sha1_v4
auth_sha1_v4_compatible
auth_aes128_md5
auth_aes128_sha1
auth_chain_a
auth_chain_b
auth_chain_c
auth_chain_d
auth_chain_e
auth_chain_f

**Obfs（Only ShadowsocksR ）:**
plain
http_simple
http_simple_compatible
http_post
http_post_compatible
tls1.2_ticket_auth
tls1.2_ticket_auth_compatible
tls1.2_ticket_fastauth
tls1.2_ticket_fastauth_compatible
