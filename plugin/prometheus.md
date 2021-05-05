# 關於 IOTA 的 prometheus Metrics


## 指標一：節點狀態： Health

```
# HELP iota_node_health Health of the node.
# TYPE iota_node_health gauge
iota_node_health 1
```

如果能用，`iota_node_health` 會是 1，不然會是零


## 指標二：是否同步：

```
iota_node_milestones{type="latest"} 62905
```

這代表最新區塊（Milestone）的編號。如上例，最新的是 62905，如果你有多個節點，而某節點的 `iota_node_milestones` 顯著低於其他節點，就是這個節點有問題。

> 其他還要補充，歡迎至 https://github.com/EasonC13/iota_playbook_zh 發 PR 喔！