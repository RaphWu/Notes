---
date: 2025-06-29
tags:
  - GUID/UUID
  - PTP
  - 分散式系統
  - 資料庫
author: Jeffrey,黑暗執行緒
sourceurl: https://blog.darkthread.net/blog/ptp/
---

前幾天提到以毫秒(ms)級 Unix 時間戳為主體，結合隨機位元，[兼顧唯一性與有序性的 UUIDv7](https://blog.darkthread.net/blog/uuidv7/)，非常適合資料庫索引與分散式系統。

既然 UUIDv7 需要靠時間確保順序，分散系統各主機時鐘的絕對精準就變得極其重要，不然你慢一毫秒，我快兩毫秒，搞到後產生的 UUID 排在前面，不就失去原本的設計精神。

有多位讀者不約而同提到一個我沒學過的新名詞 - PTP。我趁機補充了新知，順手分享學習心得。

參考：

- [網路時間標準NTP、PTP是什麼？Meta為何引進PTP？有助於元宇宙嗎？](https://www.bnext.com.tw/article/72776/is-ntp-retiring-meta-announced-that-it-will-introduce-a-new-network-time-standard-ptp-for-the-metaverse)
- [Perplexity 整理：比較 NTP 與 PTP](https://www.perplexity.ai/search/jie-shao-ji-bi-jiao-ntp-yu-ptp-Jh58az9aTPONHPerEm8GXA)

###### [NTP (Network Time Protocol)](https://zh.wikipedia.org/zh-tw/%E7%B6%B2%E8%B7%AF%E6%99%82%E9%96%93%E5%8D%94%E5%AE%9A)

NTP 大家應該已經很熟悉了，它是 1980 年代就存在的電腦系統時鐘同步協定，是最古老且廣泛使用的網路協定之一。NTP 透過 UDP 123 埠運作，採用分層（Strata）架構，從最上層的高精度時鐘（如原子鐘或 GPS）到下層的伺服器與用戶端。NTP 客戶端會定期與一個或多個 NTP 伺服器通訊，計算時間偏移與往返延遲，並根據這些資訊調整本地時鐘。

NTP 主要特點：

- 精確度：在網際網路上可達 1~50 毫秒，在區域網路可更精確。
- 部署簡單、成本低，支援所有主流作業系統與網路設備。
- 適合大規模、一般性應用（如伺服器、工作站、企業網路等）。
- 透過軟體計算時間戳記，容易受網路延遲與非對稱路徑影響。

###### [PTP (Precision Time Protocol)](https://en.wikipedia.org/wiki/Precision_Time_Protocol)

PTP（精確時間協定）定義於 IEEE 1588 標準，專為需要高精度時鐘同步的應用設計。PTP 主要用於區域網路（LAN），可達到微秒甚至納秒級的同步精度。PTP 透過主時鐘（Grandmaster）與從時鐘（Slave）之間交換時間戳記，每個PTP網域內只有一個主時鐘（Grandmaster Clock），主時鐘設備 (下圖是網路找的產品照片) 可直接收衛星系統時間 (若於封閉室內，GPS 需靠窗或從戶外接收)，結合硬體時間戳記功能，大幅減少軟體與網路延遲造成的誤差。

![[PTP_Fig2.jpg|[照片來源](https://www.oscilloquartz.com/en/products-and-services/osa-5410-series)]]

PTP 主要特點：

- 精確度：可達亞微秒（Sub-Microsecond）甚至納秒（Nanosecond）級別。
- 多用於軍事、工業自動化、金融交易、電信、能源管理等對時序要求極高的場景。
- 需專用硬體支援（如支援 PTP 的網卡或交換器），部署複雜度較高。
- 適合控制網路環境、低延遲且對同步精度有嚴格要求的應用。

| 項目           | NTP                                  | PTP                                                   |
| -------------- | :----------------------------------- | :---------------------------------------------------- |
| 精確度         | 毫秒（ms）級，區域網路可達微秒級     | 微秒（μs）至納秒（ns）級                              |
| 適用範圍       | 一般企業 IT、伺服器、工作站、IPTV 等 | 金融交易、電信、工控、能源、音視頻同步等              |
| 部署複雜度     | 簡單，軟體即可實現，無需特殊硬體     | 較高，通常需硬體支援（如交換器、網卡）                |
| 成本           | 低，僅需軟體與網路即可               | 較高，需專用硬體與網路環境                            |
| 抗網路延遲能力 | 易受網路延遲、非對稱路徑影響         | 透過硬體時間戳記大幅降低網路與設備延遲影響            |
| 標準/協定      | RFC 5905（NTPv4）                    | IEEE 1588（PTP v2/v2.1）                              |
| 典型應用       | 系統日誌、郵件伺服器、一般網路設備   | 高頻交易、電信基站、工業控制、音視頻同步、UUIDv7 產生 |

###### 簡單試玩

由以上資訊可知，PTP 的精準度很迷人，不過需要專門的硬體配合，大多會建置在資料中心或機房，不是尋常百姓家會出現的東西，想實際玩到不容易，所以我改用 Azure Linux VM 把玩一下。

除了機房端要有主時鐘，交換器要支援 PTP，個人電腦或伺服器要啟用 PTP 也要有支援的硬體，目前主是靠網卡支援 IEEE 1588 標準，以 Linux 為例，可透過 ethtool 指令檢查網卡否支援 PTP。

做法是先用 `ip link show` 列舉主機上的網卡介面，再使用 `sudo ethtool -T <interface_name>` 顯示時間戳支援狀況。

```
> ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 7c:1e:52:cc:cc:cc brd ff:ff:ff:ff:ff:ff
3: enP19555s1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master eth0 state UP mode DEFAULT group default qlen 1000
    link/ether 7c:1e:52:cc:cc:cc brd ff:ff:ff:ff:ff:ff
    altname enP19555p0s2
> sudo ethtool -T enP19555s1
Time stamping parameters for enP19555s1:
Capabilities:
        hardware-transmit
        hardware-receive
        hardware-raw-clock
PTP Hardware Clock: 1
Hardware Transmit Timestamp Modes:
        off
        on
Hardware Receive Filter Modes:
        none
        all
```

結果顯示網路介面 enP19555s1 可支援 PTP，並包含以下能力：

- hardware-transmit：支援在封包傳送時由硬體產生時間戳。
- hardware-receive：支援在封包接收時由硬體產生時間戳。
- hardware-raw-clock：支援存取硬體原始時鐘 (通常是 PHC, PTP Hardware Clock)。
- PTP Hardware Clock: 1，此網卡註冊的 PTP 硬體時鐘編號為 1。(系統中每個支援 PTP 的裝置會有一個唯一的編號，通常對應 /dev/ptp1 裝置檔案。)
- Hardware Transmit Timestamp Modes: 支援 off/on 模式
- Hardware Receive Filter Modes: 支援 none/all

想知道電腦是否支援 PTP，還有更快的做法是 `ls /dev/ptp*`，若有出現 /dev/ptp0、dev/ptp1 代表有偵測有支援 PTP 的硬體。而另個快速找出哪張網卡支援 PTP 的做法是用 `find /sys/class/net/*/device/ptp` 搜索：

![[PTP_Fig3.png]]

(註：網卡對應到 /dev/ptp1，另外還有個 /dev/ptp0，使用 `ls -l /sys/class/ptp` 查發現它指向 /sys/devices/virtual/ptp/ptp0，`cat /sys/devices/virtual/ptp/ptp0/clock_name` 顯示為 hyperv，此為 [Azure Linux VM 自帶的 PTP 時鐘來源](https://learn.microsoft.com/zh-tw/azure/virtual-machines/linux/time-sync?WT.mc_id=DOP-MVP-37580#check-for-ptp-clock-source))

接下來來試跑一下 PTP ：(註：雲端主機的電腦時鐘基本上都是準確的，不需自己動手，我純粹是想體驗自幹的情境)

```
sudo apt install linuxptp
/usr/sbin/ptp4l -v
sudo /usr/sbin/ptp4l -i enP19555s1 -m
```

實測結果不如人意，由於網卡在 VM 環境沒有接到來自主時鐘的 Announce 訊息，故只能以本地時鐘為準，enP19555s1 也自行宣佈登基，成為 PTP 網路的 Grandmaster。

![[PTP_Fig4.png]]

從網路找到[實例](https://docs.fedoraproject.org/en-US/fedora/f40/system-administrators-guide/servers/Configuring_PTP_Using_ptp4l/)增長見聞，結束這回合。

```
~]# ptp4l -i em3 -m
selected em3 as PTP clock
port 1: INITIALIZING to LISTENING on INITIALIZE
port 0: INITIALIZING to LISTENING on INITIALIZE
port 1: new foreign master 00a069.fffe.0b552d-1
selected best master clock 00a069.fffe.0b552d
port 1: LISTENING to UNCALIBRATED on RS_SLAVE
master offset -23947 s0 freq +0 path delay       11350
master offset -28867 s0 freq +0 path delay       11236
master offset -32801 s0 freq +0 path delay       10841
master offset -37203 s1 freq +0 path delay       10583
master offset  -7275 s2 freq -30575 path delay   10583
port 1: UNCALIBRATED to SLAVE on MASTER_CLOCK_SELECTED
master offset  -4552 s2 freq -30035 path delay   10385
```

Windows 也支援 PTP 嗎？Yes! 從 [Windows Server 2019 / Windows 10 1809+ 已加入支援](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/insider-preview?WT.mc_id=DOP-MVP-37580#precision-time-protocol)，因此只要設備到位或是用雲端主機，開發人員可假設每一台主機的時鐘都會是準的就好，不需操煩。
