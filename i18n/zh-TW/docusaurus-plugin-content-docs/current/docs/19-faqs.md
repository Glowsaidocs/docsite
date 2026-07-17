---
id: faqs
sidebar_position: 19
---

# Glows.ai GPU 租賃平台常見問題

本文整理 Glows.ai GPU 租賃平台的常見問題，涵蓋資料儲存、執行個體環境、連線方式、自動部署、計費與服務保障等內容。若您需要進一步協助，歡迎透過官方支援管道與我們聯絡。

## DataDrive 與 Snapshot 資料管理

### 1. 是否支援關閉執行個體後停止運算費用，只保留儲存費用？

支援。當您暫時不需要執行訓練或推論任務時，可以釋放執行個體（Instance）以停止 GPU 運算資源計費，並透過 DataDrive 與 Snapshot 保留需要的資料或環境。

- 模型、資料集、程式碼等資料型檔案，建議存放於 DataDrive。DataDrive 支援在不啟動執行個體的情況下上傳與下載資料。
- Python 套件、系統軟體、環境設定等需要下次沿用的內容，建議在釋放執行個體前按一下 `Take Snapshot` 建立環境快照。
- Snapshot 會保存 DataDrive 以外的執行個體內部變更。Snapshot 越小，後續從快照建立新執行個體通常越快。

DataDrive 與 Snapshot 需在 Storage Space 頁面購買儲存方案。您可依實際需求分配 DataDrive 與 Snapshot 的容量。

### 2. 哪裡可以看到已上傳到 DataDrive 的檔案？

建立執行個體時選擇掛載 DataDrive。登入執行個體後，即可在 `/datadrive` 目錄查看檔案。

```bash
ls /datadrive
```

### 3. 多台執行個體可以共用同一個 DataDrive 嗎？

可以。同一區域內的執行個體可透過掛載 DataDrive 共用資料。建議在正式任務前先建立測試執行個體，確認映像檔環境、路徑與權限都符合您的工作流程。

### 4. 執行個體釋放或租期結束後，執行個體硬碟上的資料會消失嗎？

會。執行個體硬碟僅在執行個體運作期間有效，釋放後資料會被清除。

建議將重要資料存放於 DataDrive；如需保留環境或已安裝軟體，請在釋放執行個體前建立 Snapshot。若不使用平台儲存，也可在釋放前透過 `scp` 等方式下載至本機。

### 5. DataDrive 與 Snapshot 會在儲存方案到期後自動刪除嗎？

不會立即自動刪除。DataDrive 與 Snapshot 採儲存方案管理；方案到期後，資料會保留一段可恢復狀態，但需續費後才能繼續正常使用。為避免影響任務，建議在方案到期前完成續費。

### 6. 已掛載 DataDrive 的執行個體重開機後，`/datadrive` 目錄消失，資料會遺失嗎？

不會。DataDrive 資料不會因執行個體重開機而遺失。若 VM重開機後未自動掛載，可嘗試在執行個體內執行以下指令手動掛載：

```bash
sudo mount -t virtiofs DataDrive /datadrive
```

此情況通常是重啟過程中未觸發自動掛載。平台會持續優化相關流程。

### 7. 如何更有效率地使用 Snapshot？

Snapshot 會保存執行個體硬碟上的變更。建議將環境、套件與系統設定保留在執行個體硬碟，將資料集、模型權重與程式碼放在 DataDrive。這樣可以降低 Snapshot 大小，並提升後續從快照建立新執行個體的速度。

跨區使用 Snapshot 時，建立新執行個體的速度會受 Snapshot 大小與網路傳輸狀況影響，因此建議保持 Snapshot 精簡。

### 8. DataDrive 網頁版可以一次上傳整個資料夾嗎？

目前 DataDrive 網頁版主要支援單一檔案上傳。若需要上傳整個資料夾，建議先壓縮為單一檔案後再上傳，或改用 DataDrive PC 版。

DataDrive PC 版下載連結：[https://glows.ai/datadrive](https://glows.ai/datadrive)

### 9. 在 DataDrive 網頁版上傳大檔失敗怎麼辦？

建議改用 DataDrive PC 版。PC 版支援 Windows 與 macOS，適合上傳或下載大型檔案與資料夾。

下載連結：[https://glows.ai/datadrive](https://glows.ai/datadrive)

### 10. 不同區域的 DataDrive 可以直接同步嗎？

目前不支援跨區 DataDrive 直接同步。若需將資料從 TW-01 移轉至 TW-02，請先下載到本機，再上傳至目標區域的 DataDrive。

### 11. Snapshot 可以匯出嗎？

不支援。Snapshot 為 Glows.ai 平台內部使用的快照格式，目前不支援直接匯出。

### 12. 可以不啟動執行個體就上傳或下載資料嗎？

可以。DataDrive 支援離線上傳與下載，不需要先啟動 GPU 執行個體。

- DataDrive 教學影片：[Datadrive](https://www.youtube.com/watch?v=_MtYfNKQ3xA)
- DataDrive PC 版：[Download](https://glows.ai/datadrive)

### 13. 已啟動的 VM 可以新增或掛載 DataDrive 嗎？

目前不支援在已啟動的執行個體上新增掛載 DataDrive。請在建立執行個體時選擇需要掛載的 DataDrive。

### 14. DataDrive 已清空，為什麼 Storage 頁面仍顯示有容量被使用？

透過 DataDrive 介面刪除資料後，Storage 頁面通常需要 5-10 分鐘同步狀態。若需確認實際使用量，可建立同區域執行個體並掛載 DataDrive，然後執行：

```bash
# 查看總儲存使用量
df -h | grep datadrive

# 查看 /datadrive 下各檔案與資料夾
ls -alh /datadrive
```

Ubuntu 檔案系統可能會將部分刪除的檔案暫存至 `/datadrive/.Trash-0`。若確認不再需要，可執行以下指令永久刪除：

```bash
# 此操作無法復原，請確認路徑無誤後再執行
rm -rf /datadrive/.Trash-0
```

### 15. 寫入 DataDrive 時出現 `Disk quota exceeded` 是什麼原因？

此錯誤通常表示儲存空間不足，常見原因包括：

1. DataDrive 容量已使用完畢，需清理不需要的檔案，或在 Storage 頁面擴充容量。
2. Storage 方案已到期，需續費後才能繼續使用。

### 16. Storage 續費時如何降級方案？

若希望從較高容量方案降級至較低容量方案，需先確保目前實際使用量低於目標方案容量。

例如原方案為 300GB，若要續費為 200GB，需先將實際使用量清理至 200GB 以下。可刪除 DataDrive、Snapshot 或 Images 中不再需要的檔案。

![Storage usage before cleanup](../../../../../docs/docs-images/p20faqs/01.png)

清理後確認使用量已低於目標方案容量。

![Storage usage after cleanup](../../../../../docs/docs-images/p20faqs/02.png)

按一下 `Modify`，將已分配總額度調整至小於或等於目標方案容量，然後按一下 `Update`。

![Modify storage allocation](../../../../../docs/docs-images/p20faqs/03.png)

完成後按一下 `Upgrade`，即可選擇較低容量方案。

![Select lower storage plan](../../../../../docs/docs-images/p20faqs/04.png)

### 17. 檔名後綴 `.partial` 是什麼意思？

`.partial` 是系統在檔案上傳期間產生的暫存檔。上傳完成後會自動消失，不影響正常使用。


## 映像檔與儲存空間管理

### 1. 如何上傳自訂 Docker 映像檔？

請參考官方教學：[Upload Custom Docker Image](https://docs.glows.ai/docs/upload-custom-docker-image)

### 2. Image 檔案過大，網路不穩或切換頁面導致上傳中斷怎麼辦？

平台支援斷點續傳。即使上傳過程中斷，也可以從中斷位置繼續上傳，不需要重新開始。



### 3. 如何將本機模型與 Python 程式碼傳入 Glows.ai 執行個體？

建議優先使用 DataDrive：

1. 將本機檔案上傳至 DataDrive。
2. 建立執行個體時掛載 DataDrive。
3. 登入執行個體後在 `/datadrive` 目錄讀取檔案。

若執行個體已啟動，也可以使用 `scp` 傳輸。以下為 Windows 本機傳輸至執行個體 `/home` 的範例：

```bash
scp -P 23675 -r C:\Users\Data root@tw-03.access.glows.ai:/home
```

每台執行個體的 SSH 連線地址與連接埠可能不同，請以執行個體頁面顯示資訊為準。

### 4. Glows.ai 的快照功能會保留已安裝程式與資料嗎？

會。Snapshot 會保存建立快照當下執行個體內的檔案、設定、已安裝軟體與組態內容，但不包含 DataDrive 內的資料。DataDrive 資料本身儲存在雲端，不需要重複保存。

## 映像檔與應用操作

### 1. 使用 FramePack 等工具產生影片後無法下載，該怎麼處理？

請先確認執行個體尚未釋放，然後嘗試透過 JupyterLab 手動下載：

1. 進入該執行個體的 JupyterLab。
2. 前往 `/FramePack/outputs/`。
3. 找到產生的 `.mp4` 檔案。
4. 在檔案上按一下右鍵，選擇 `Download`。

若仍無法下載，請截圖並聯絡平台技術支援。

### 2. ComfyUI 匯入工作流程後缺少自訂節點，如何處理？

請先建立執行個體並載入工作流程，然後提供錯誤畫面或缺少節點的提示。平台技術支援可協助確認需要安裝的自訂節點或相依套件。

### 3. ComfyUI 測試 WAN 2.1 時提示缺少 Model，代表什麼？

通常表示模型尚未正確載入。請確認模型檔案位置、工作流程設定與模型名稱是否正確；如仍無法解決，請提供錯誤截圖供技術支援分析。

## 執行個體環境、權限與連線

### 1. 是否支援 Docker 或 K8s？VM 與 Container 有什麼差異？

Glows.ai 提供 VM 與 Container 兩種底層雲端服務型態。

- Container 執行個體提供較多預先設定好的基礎環境，環境搭配較簡單，但不支援在執行個體內自行使用 Docker，也不支援修改 NVIDIA 驅動程式等底層軟體。
- VM 執行個體具備較高權限，支援自訂 GPU 驅動程式、Docker、Systemctl 等能力，適合需要完整系統控制權的工作負載。

建立執行個體時選擇 `Windows` 或 `For VM` 類型 Image，即為 VM 執行個體。

![VM image selection](../../../../../docs/docs-images/p20faqs/05.png)

### 2. 是否具備 root 權限？

具備。執行個體預設使用 root 帳號登入。您也可以登入後自行建立其他使用者。

### 3. 是否支援自選 OS 環境？

目前平台提供 Ubuntu 系列 OS 環境與 Windows Server 2025。一般情境建議優先選擇 Ubuntu。若有其他 OS 需求，可聯絡我們提出需求評估。

### 4. Windows 環境是否可能出現遠端操作卡頓或不穩定？

Windows 在遠端桌面操作下可能受網路、圖形介面與工作負載影響而出現卡頓。若任務不依賴 Windows，建議優先使用 Linux 環境，以取得更穩定的訓練與推論體驗。

### 5. 支援哪些遠端連線方式？

Ubuntu 執行個體支援 SSH 與 JupyterLab；Windows 執行個體支援 RDP。建議優先使用平台提供的連線方式。若需要其他連線工具，可登入執行個體後自行安裝相關服務。

### 6. RDP 連線網址中的 `tw-02.access.glows.ai` 是否代表執行個體所在區域？

不一定。連線網址中的 `tw-02` 可能是存取節點代碼，不代表實際運算主機所在區域。請以執行個體頁面顯示的部署區域為準。

### 7. 是否可以即時查看 GPU/CPU 使用率、記憶體與硬碟等資訊？

可以。執行個體建立後，執行個體頁面提供 Monitor 功能，可查看 CPU/GPU 使用率、GPU 記憶體、系統記憶體、硬碟空間等指標走勢。

![Instance monitor](../../../../../docs/docs-images/p20faqs/06.png)

### 8. NVIDIA Driver 可以由客戶自行更新嗎？

視執行個體類型而定：

1. Container 類型執行個體暫不支援自行更換 NVIDIA Driver。
2. VM 類型執行個體可由客戶自行下載並更新 Driver；如有需要，平台可提供技術支援。[Contact Us](https://docs.glows.ai/docs/contact-us)

![NVIDIA driver example](../../../../../docs/docs-images/p20faqs/07.png)

## 自動部署、網路與企業功能

### 1. 是否支援自動擴縮？

支援。平台提供 Autodeploy 功能，可在收到請求時自動建立執行個體、啟動服務，並在閒置後釋放執行個體。

典型流程如下：

1. 客戶請求 Autodeploy 連結。
2. Glows.ai 後端依設定建立執行個體。
3. 執行您設定的 Service Start Command。
4. 將請求轉發至您設定的服務連接埠。
5. 回傳處理結果。
6. 若連續 n 分鐘內沒有新的請求，平台會自動釋放執行個體。

另外也支援客戶透過 Glows.ai SDK，以程式化方式自訂控制執行個體的啟動與釋放。
Autodeploy 使用教學：[Autodeploy](https://docs.glows.ai/docs/auto-deploy-usage)
SDK 使用教學：[SDK Docs](https://sdkdoc.glows.ai)

### 2. 是否提供 CLI 或 API 自動部署？

企業客戶如有 API 自動部署需求，可聯絡我們申請評估。SDK 使用教學：[SDK Docs](https://sdkdoc.glows.ai)


### 3. 是否可透過 API 開關機器以避免閒置費用？

API 功能目前主要提供給具備自動化需求的企業客戶。如需使用，請透過官方支援管道洽詢。

### 4. 平台是否販售網路流量？是否提供固定頻寬或固定 IP？

一般情況下，平台不另外收取資料傳輸費。若需要固定 1Gbps 頻寬、靜態 IP 等進階網路能力，可申請加購，由業務與技術團隊確認需求後啟用。

### 5. 執行個體關機或重啟後 IP/DNS 會變更嗎？

可能會。若需要更穩定的服務入口，建議使用 Autodeploy 固定服務連結，並由您自行設計流量分配策略。

### 6. 是否可使用企業私有網段或 VPN 限定存取？

相關功能正在開發與規劃中。未來企業客戶可望設定允許存取執行個體的網路 IP，例如僅允許指定 VPN 網段透過 SSH 存取。正式可用時間請以平台公告為準。

### 7. 既有伺服器或 GPU 機器可以整合進 Glows.ai 嗎？

可透過混合雲或私有雲方案進行專案評估，將本地 CPU/GPU 機器整合為運算資源池的一部分。具體可行性需由技術團隊根據硬體、網路與部署需求確認。

如有需求，請聯絡我們：[Contact Us](https://docs.glows.ai/docs/contact-us)

### 8. 私有雲是否支援 CPU 機器加入並統一管理？

支援。私有雲方案可依需求整合本地非 GPU 機器，並提供統一管理與調度能力。具體方案需經專案評估。

## GPU 資源與底層支援

### 1. H100/H200 GPU 是否支援 NVIDIA MIG 切分？

Glows.ai 平台部署的 H100/H200 GPU 支援 NVIDIA MIG 能力。具體開放方式與設定需依產品方案與實際資源情況確認。

### 2. GPU 是否都是整片租賃？

目前平台主要提供整片 GPU 租賃。

### 3. GPU 之間是否支援 NCCL 通訊？

支援。NCCL 可用於分散式訓練與多 GPU 資料同步。

## 帳號、計費與儲值

### 1. 是否可以使用一般 Email 註冊，而不是 Gmail？

目前系統預設支援 Gmail 註冊。若您需要使用一般 Email 建立帳號，請聯絡技術或業務團隊協助評估與處理。

### 2. 線上儲值是否有最低金額？

有。平台線上儲值最低金額為 10 美元，並以 10 美元為遞增單位。

### 3. 忘記儲值導致點數耗盡怎麼辦？

點數耗盡後，系統會自動為執行中的執行個體建立快照並關閉執行個體。重新儲值後，您可從自動產生的快照恢復環境。

### 4. 是否有最低租用時數？

沒有。平台採用 pay-as-you-go 計費模式，用多少付多少。執行個體未運作時不產生 GPU 運算費用，僅依實際使用情況收取儲存費用。

### 5. H200 GPU 價格是多少？

H200 GPU 隨用隨付價格請以平台頁面顯示為準。長期租用或包月需求可聯絡我們洽談優惠方案。

### 6. 儲存空間如何計價？

儲存空間採方案或用量方式計費，具體價格請以平台 Storage 頁面顯示為準。大量採購或企業需求可聯絡我們洽談。

### 7. 是否有大量或長期租用優惠？

有。大量或長期租用請聯絡我們：[Contact Us](https://docs.glows.ai/docs/contact-us)

### 8. 使用前是否需要支付訂閱費、設定費或最低使用費？

不需要。一般使用情境無訂閱費、設定費或最低使用費。

### 9. 軟體或平台合作客戶如何計價？

可依使用量、儲存量與合作模式評估計費方案。若涉及代理、轉售或平台合作，可聯絡業務團隊洽談合作協議。

## 服務保障與補償

### 1. 是否提供 SLA？

提供。平台目前主要針對硬體與儲存服務提供服務水準承諾：

- 核心硬體服務：99% 可用性。
- DataDrive 儲存服務：99.9% 可用性。

具體條款以正式服務協議為準。

### 2. 若服務中斷，是否有備援與補償政策？

若因硬體或平台原因導致租用的執行個體不可用，我們會儘快核查故障時間與原因，優先恢復服務與資料，並依情況提供補償。

基礎補償參考：

- 故障 1-3 小時：補償 6 小時。
- 故障 6-12 小時：補償 12 小時。

若故障造成資料遺失、任務中斷且無法恢復等重大影響，平台會依具體情況進一步評估。除物理硬碟故障等特殊情況外，正常已存資料通常不會遺失。重要資料建議優先存放於 DataDrive。
