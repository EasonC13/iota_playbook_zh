# 如何建立新版本的 IOTA Hornet 節點？


## 1. 安裝或更新至最新版本

```
sudo sh -c 'echo "deb http://ppa.hornet.zone stable main" >> /etc/apt/sources.list.d/hornet.list'
sudo apt update
sudo apt install hornet
```

> 如你是從舊版 Hornet (0.5.x) 更新，且像我一樣使用其他硬碟掛接到 `/var/lib/hornet`，強烈建議你先把 `/var/lib/hornet` 內的檔案移動到其他地方備份好，或於更新詢問是否幫你備份時選擇 No，否則在更新時他會自動幫你把 `/var/lib/hornet/*` 的內容移動到 `/var/lib/hornet_backup`，之後我的系統碟就因為空間不足炸掉了。

## 2. 允許程式從系統啟動

```
sudo systemctl enable hornet.service
```

## 3. 啟動節點

```
sudo service hornet start
```

## 4. 管理節點狀態

### 即時日誌監控：

```
journalctl -fu hornet
```

### 關閉節點：

```
sudo systemctl stop hornet
```

### 重啟節點：

```
sudo systemctl restart hornet
```


> 強烈建議加 alias 到你的 .bashrc
> 節省時間的好幫手：
> alias iotast="journalctl -fu hornet"
> alias iotarestart="sudo systemctl restart hornet"
> alias iotastop="sudo systemctl stop hornet"

## 進入管理儀表板（Dashboard）

### 設定帳號密碼

在進去 Dashboard 前，你需要先設定密碼。使用主機終端機輸入以下指令來產生密碼的 Hash 與 Salt。

```
hornet tool pwdhash
```

遵循指令，輸入兩次要設定的密碼後即可完成，之後你會看到類似下方的內容：

```
Success!
Your hash: 81c12baaefd2c062e78d60656f5276e717c2108a485cfe1b9ebe909b7dd86252
Your salt: b18947bc191d7f4eff64fd008a1109459b1eca2490a80796ae39064214524d2f
```

這就是你的密碼的 Hash 與 Salt，將他們放到 `/var/lib/hornet/config.json` 的 dashboard 的 auth 中，範例如下：

```
   "dashboard": {
    "bindAddress": "0.0.0.0:8081",
    "auth": {
            "enbaled": false,
      "sessionTimeout": "72h",
      "username": "admin",
      "passwordHash": "81c12baaefd2c062e78d60656f5276e717c2108a485cfe1b9ebe909b7dd86252",
      "passwordSalt": "b18947bc191d7f4eff64fd008a1109459b1eca2490a80796ae39064214524d2f"
    }
  },
```

使用者名稱也可以自行更改。

### 允許遠端連入 Dashboard

如果你的 hornet 在遠端的主機（如 VPS）上，請至 `/var/lib/hornet/config.json` 設定連線 IP 為 `0.0.0.0:8081`

```
  "dashboard": {
    "bindAddress": "0.0.0.0:8081",
```

或者你要用其他 Port 也行。甚至你也可以用 nginx（待補）。

之後重新啟動你的 Hornet 節點，即可套用設定更改。


### 前往儀表板設定

然後就可以去 http://your.ip:8081 看到 Hornet 的新版本 Dashboard 囉！

![](https://i.imgur.com/jmFgJZH.jpg)

如果你看到的跟上圖不一樣，而是跟下圖相似，別擔心，請點選左邊 Login，用先前設定的帳密登入即可

![](https://i.imgur.com/jhT9ZpM.png)


## 新增鄰居

這個版本的鄰居格式，與舊版不同，因此要重新新增。因為這個版本你的節點有自己的 Private Key 與相對應的 Public Key 跟 ID，而在新增鄰居時，需要加上鄰居的 ID。這是舊版沒有的功能。

新版鄰居格式範例如下：

Address: `/dns/your.ip.com/tcp/15600`

ID: `12D3KooWLAMAzV9rMuuFLxuxfkaoRmnq5joBZ6B8cazEoHeYQ92k`

其中你的 ID 要去 Dashboard Home 的紅色圈圈處複製

![](https://i.imgur.com/ih3Lfov.png)


### 找鄰居

在這個版本的 Hornet，目前（2021/05/05）還不支援 Auto Peering，因此你需要手動去尋找鄰居。

請你到 [IOTA 的 Discord](https://discord.iota.org/) 去找鄰居，也歡迎找我連線（請私：`EasonC13#4070`）
請至 nodesharing 頻道，貼上你在徵求鄰居節點的消息，就會有其他人來私訊你囉！
範例如下：

![](https://i.imgur.com/kcKd5Se.png)

或者你也可以私前面每一個在徵求鄰居節點的人，請他們把你加到他們的列表。
參考範例如下：

![](https://i.imgur.com/gwkhRfl.png)

> 我貼出訊息並逐一私訊後，花五小時左右，就找到十二個鄰居了，很快，就勇敢的開私吧！

## 其他新功能（待補充）

這個版本的 Hornet 支援 Prometheus 監控，只要將 `config.json` 的 `enablePlugins` 新增 `Prometheus` 即可

```
    "enablePlugins": [
      "Prometheus",
      "Spammer"
    ]
```

順便也要設定能從外網監聽狀態，將 `bindAddress` 及 `target` 從 `localhost` 改為 `0.0.0.0`

```
  "prometheus": {
    "bindAddress": "0.0.0.0:9311",
    "fileServiceDiscovery": {
      "enabled": false,
      "path": "target.json",
      "target": "0.0.0.0:9311"
    },
```

之後就可以去 http://your.ip:9311/metrics 監控你的節點狀態囉！

相關指標的意義請參考：[IOTA Hornet Prometheus 用法](https://github.com/EasonC13/iota_playbook_zh/blob/main/plugin/prometheus.md)

> 不過 Prometheus 監控 API，洩漏出的訊息基本上就跟登入後的 Dashboard 相同，他人可以存取到你現在連線的鄰居列表，因此可能會讓你的節點被攻擊的可能性增加。因此請斟酌使用。
