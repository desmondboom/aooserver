# SonoBus Aooserver 中继服务部署文档

本文档记录了部署 SonoBus 中继服务（使用 aooserver 项目）的完整过程，包含服务器环境、依赖安装、编译、部署、自启动配置、防火墙端口设置以及客户使用说明，作为交付记录和后续维护参考。

---

## 一、服务器配置

- **系统**：CentOS 7
- **公网 IP**：客户自有服务器，具备公网访问能力
- **目标服务**：只运行 aooserver 中继服务，无需其他后端或 Web 服务
- **服务端口**：UDP 10998（默认）

---

## 二、项目仓库

- **开源项目地址**：[https://github.com/essej/aooserver](https://github.com/essej/aooserver)
- **项目描述**：aooserver 是由 SonoBus 推荐的新一代中继服务器，支持 AoO 协议，适用于高效、低延迟的音频传输。

---

## 三、依赖安装

执行以下命令安装所需依赖：

```bash
sudo yum update -y
sudo yum groupinstall -y "Development Tools"
sudo yum install -y cmake git gcc-c++ liblo-devel opus-devel
```

---

## 四、项目构建

```bash
cd /opt
sudo git clone https://github.com/essej/aooserver.git
cd aooserver
sudo make
```

> 编译完成后，会生成可执行文件 `aooserver`

---

## 五、部署 & 自启动配置

### 将可执行文件移动至标准路径

```bash
sudo cp /opt/aooserver/aooserver /usr/local/bin/aooserver
```

### 创建 Systemd 服务

```bash
sudo vi /etc/systemd/system/aooserver.service
```

填入内容：

```ini
[Unit]
Description=AoO / SonoBus Relay Server
After=network.target

[Service]
ExecStart=/usr/local/bin/aooserver -p 10998
Restart=always
RestartSec=3
StartLimitIntervalSec=0
User=root
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### 启用并启动服务

```bash
sudo systemctl daemon-reexec
sudo systemctl enable aooserver
sudo systemctl start aooserver
```

---

## 六、防火墙设置

确保开放 UDP 10998 端口：

```bash
sudo firewall-cmd --permanent --add-port=10998/udp
sudo firewall-cmd --permanent --add-port=10998/tcp
sudo firewall-cmd --reload
```

---

## 七、客户端连接说明（给客户）

在 SonoBus 客户端中：

1. 设置服务器地址为服务器公网 IP
2. 输入相同房间名 → 点击 Join → 成功连接

---

## 八、后续维护建议

- 使用 `sudo systemctl status aooserver` 查看服务状态
- 使用 `journalctl -u aooserver -f` 查看实时日志
- 重启服务使用：`sudo systemctl restart aooserver`
- 如果修改了服务配置，请执行：

  ```bash
  sudo systemctl daemon-reexec
  sudo systemctl restart aooserver
  ```

---

## 九、交付确认

- 已编译部署 aooserver 至 `/usr/local/bin`
- 已配置 Systemd 自启动
- 已开放 UDP,TCP 10998 端口并测试连接成功
- 客户可直接通过 SonoBus 客户端连接

> 本服务已测试通过，符合客户用于音频中继的稳定性要求。

---

如需再次部署、迁移或加固，请联系维护方。
