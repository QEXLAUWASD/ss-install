# Shadowsocks 伺服器安裝腳本

互動式安裝 Shadowsocks-libev、ShadowsocksR 或 Shadowsocks-Rust。腳本需要 root 權限、網路連線及 `wget`；Rust 版本另需可取得對應架構的官方預編譯檔。腳本會安裝套件並修改服務、防火牆及系統設定，請在目標主機上執行。

## 安裝

```bash
wget -O shadowsocks.sh https://raw.githubusercontent.com/QEXLAUWASD/ss-install/master/shadowsocks.sh
chmod +x shadowsocks.sh
sudo ./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

在選單輸入 `1` 安裝 Shadowsocks-libev、`2` 安裝 ShadowsocksR、`3` 安裝 Shadowsocks-Rust。Rust 安裝流程會依序詢問密碼、主要端口、額外端口數量、UDP 支援及加密方法。額外端口數量留空時為 `0`；例如主要端口是 `10223`，輸入 `4` 會建立 `10224`、`10225`、`10226`、`10227`。若連續端口會超過 `65535`，安裝器會要求重新輸入。所有端口共用密碼、加密方法和同一個 `ssserver` 程序。

安裝後按輸出內容設定客戶端的伺服器位址、端口、密碼及加密方法。每個客戶端連線仍需指定端口；如需跨端口分流，請在客戶端或外部負載平衡器設定。CentOS 安裝流程會嘗試開放所選端口；Debian、Ubuntu 及雲端防火牆需要自行開放相應的 TCP 端口，啟用 UDP 時也要開放 UDP 端口。

## Shadowsocks-Rust 服務與設定

主配置位於 `/etc/shadowsocks-rust/config.json`。使用 systemd 的主機可執行：

```bash
sudo systemctl status ssserver
sudo systemctl restart ssserver
```

使用 SysV init 的主機可執行 `sudo service shadowsocks-rust status` 或 `sudo service shadowsocks-rust restart`。Rust 安裝器會寫入 `/etc/sysctl.d/90-shadowsocks-rust.conf`；若內核支援 BBR，會啟用 BBR 與 `fq`。檢查目前生效的網路設定；第二個指令適用於 systemd 主機：

```bash
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc
systemctl show ssserver -p LimitNOFILE
```

主服務及下述 systemd 實例的 `LimitNOFILE` 設為 1048576；SysV 啟動腳本嘗試設為 65535。實際值仍受主機或容器上限限制。

### 獨立實例（systemd）

若要用不同程序服務另一組端口，先建立 `/etc/shadowsocks-rust/edge.json`。可從主配置複製，再修改 `server_port`，確保它不與主服務或其他實例重複。將配置授予服務帳戶讀取權限後啟動：

```bash
sudo chgrp "$(id -gn nobody)" /etc/shadowsocks-rust/edge.json
sudo chmod 640 /etc/shadowsocks-rust/edge.json
sudo systemctl enable --now ssserver@edge
sudo systemctl status ssserver@edge
```

模板讀取 `/etc/shadowsocks-rust/<名稱>.json`。若複製的主配置含多個 `servers`，請逐一修改或刪除衝突端口。獨立實例的端口也需自行加入防火牆。停止實例可執行 `sudo systemctl disable --now ssserver@edge`。

## 其他命令

```bash
sudo ./shadowsocks.sh uninstall  # 在選單中選擇已安裝的版本
sudo ./shadowsocks.sh upgrade    # 目前僅支援 Shadowsocks-libev
```

Rust 解除安裝會移除安裝器建立的 sysctl 檔案和 systemd 模板。解除安裝前請先停止額外實例；執行中的 sysctl 值可能要到重新開機後才恢復。

Shadowsocks-libev：`/etc/init.d/shadowsocks-libev start|stop|restart|status`，配置位於 `/etc/shadowsocks-libev/config.json`。

ShadowsocksR：`/etc/init.d/shadowsocks-r start|stop|restart|status`，配置位於 `/etc/shadowsocks-r/config.json`。

## 加密方法與協定參考

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
