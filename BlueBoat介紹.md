# BlueBoat 系統技術與架構說明書

* **載具全名**：BlueRobotics BlueBoat USV
* **原廠廠商**：BlueRobotics（美國）
* **載具定位**：開源、輕量化、模組化小型雙體水面無人載具（USV）

## 一、 背景與起源

* **突破高昂門檻**：  
早期水面測繪與無人載具多為動輒百萬的高單價封閉式系統。BlueRobotics 延續其開源水下無人機（BlueROV2）的成功經驗，推出兼具高性價比與高度改裝彈性的水面標準平台。
* **面向開源與學術生態**：  
原生深度整合全球最大的開源無人載具生態系 **ArduPilot** 與 **BlueOS**，降低自主航行演算法的落地難度。
* **核心應用場景**：  
廣泛應用於水深測量、環境水質監測、學術研究驗證，以及國際水面無人載具競賽。

## 二、 系統硬體與組成架構 (Composition)

BlueBoat 由機械結構、推進動力、電控飛控與軟體通訊四大子系統組成：

### 1. 機械與船體結構 (Mechanical & Hull)

| 組件名稱 | 規格與結構特性 | 功能說明 |
| --- | --- | --- |
| **雙體船身 (Catamaran)** | 左右獨立高密度聚乙烯/複合材料浮筒 | 提供優異的初穩度與水面抗浪性，航行吃水淺、阻力小。 |
| **鋁合金橫樑 (Crossbars)** | 具備標準 T-track 軌道槽，可折疊拆裝 | 連接兩側浮筒，並提供充足的甲板空間以鎖附光達、相機或天線支架。 |
| **水密艙體 (Watertight Hatches)** | 雙 O 型環（O-ring）密封防水蓋 | 保護內部電控板、電池與機載電腦免於海水/淡水滲入。 |

---

### 2. 動力與推進系統 (Propulsion & Power)

* **無刷推進器**：  
配置兩顆 **M200 / T200** 水下無刷推進器，懸掛於左右浮筒後下方。
* **差速轉向控制**：  
船體無實體方向舵，透過左右兩側推進器的轉速差與正反轉解算達成原地旋轉與轉向。


* **電源供應模組**：  
支援 4S 鋰電池組（XT90 / XT60 接頭），搭配專用電源管理模組（Power Module / BEC），將動力電壓分流並降壓提供飛控板與機載電腦純淨電源。

---

### 3. 電控與核心硬體 (Avionics & Electronics)

```
 [ 動力鋰電池 ] ──▶ [ 電源管理模組 (BEC) ]
                           │
     ┌─────────────────────┴─────────────────────┐
     ▼                                           ▼
[ Raspberry Pi (機載運算電腦) ] ── (內部序列通訊) ──▶ [ Navigator 飛控板 ]
     │                                           │
  (BlueOS 系統)                                (ArduPilot 韌體)
     │                                           │
[ Wi-Fi / 數傳電台 / 乙太網 ]               [ 雙推進器電變 (ESC) + IMU / 羅盤 ]

```

* **Navigator 飛控板**：  
內建雙 IMU、磁力計、氣壓計，提供精確姿態估算與 PWM 馬達控制輸出。
* **Raspberry Pi 機載電腦**：  
作為核心主機，與 Navigator 飛控板直連，負責系統開機、網路路由與外部擴充模組調度。
* **感測與定位套件**：  
外置整合式 GPS / 羅盤蘑菇頭天線，提供公尺級或 RTK 公分級衛星定位與絕對磁航向。

---

### 4. 軟體與通訊架構 (Software & Protocol Stack)

* **ArduPilot (Rover 韌體)**：  
負責底層感測器濾波（EKF）、航向姿態控制、航點導航（Waypoint Navigation）與各類失聯返航（Fail-Safe）保護機制。


* **BlueOS 管理系統**：  
運行於樹莓派上的 Linux 管理介面，提供 Web 儀表板，支援韌體更新、感測器校準、MAVLink 路由轉發與 Docker 插件管理。


* **MAVLink 通訊協定**：  
系統內部與對外的標準二進位通訊協定，所有姿態、航速、GPS 坐標與馬達推力指令皆透過 MAVLink 封包於飛控、地面站（QGC）與上層演算法間傳遞。

## 三、 自主演算法擴充能力 (Autonomous Integration)

BlueBoat 原生架構具備極高的擴充性，可透過 MAVLink 端點直接與高階決策框架對接：

1. **ROS 2 整合**：
* 機載電腦（如 NVIDIA Jetson 或 Mini-PC）運行 ROS 2，透過 `MAVROS2` 或 `Micro-XRCE-DDS` 接收載具狀態，並輸出 `cmd_vel` 速度指令給 ArduPilot。
* 適合掛載 LiDAR 與相機進行水面 SLAM 建圖、障礙物辨識與 Nav2 路徑規劃。


2. **MOOS-IvP 整合**：
* 透過開源橋接模組（`iMOOS2MAVLink` / `iArduPilot`），由 MOOS-IvP 的 `pHelmIvP` 進行多目標行為決策（如避碰規則 COLREGS），直接下發 `DESIRED_HEADING` 與 `DESIRED_SPEED` 指令控制船體。