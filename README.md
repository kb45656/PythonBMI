# BMI 計算器

使用 Flask 製作的成人 BMI 網頁計算器。輸入身高（公分）和體重（公斤）後，頁面會顯示 BMI（小數點後一位）、體位分類與生活建議，也可以依身高與 BMI 試算兵役體位。

## 啟動

需安裝 Python 3.9 以上版本。在專案資料夾執行：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python app.py
```

接著開啟 http://127.0.0.1:5000/ 。若 PowerShell 不允許啟用虛擬環境，也可以使用 `.\.venv\Scripts\python.exe -m pip install -r requirements.txt` 和 `.\.venv\Scripts\python.exe app.py`。

## 計算方式

BMI = 體重（公斤）÷ 身高（公尺）²。分類採臺灣成人體位參考標準：低於 18.5 為體重過輕、18.5 至未滿 24 為健康體位、24 至未滿 27 為體重過重、27 以上為肥胖。頁面以顯示至小數點後一位的 BMI 進行分類。

這只是成人體位參考，不能取代專業醫療評估，也不適用於兒童、青少年或孕婦。

## 兵役體位試算

「判斷兵役體位」會依內政部役政司的身高與 BMI 規則進行初步試算：

- 身高 154 公分以下或 196 公分以上：免役體位。
- 身高 155 至 157 公分：BMI 15 至 35 為替代役體位，超出範圍為免役體位。
- 身高 158 至 195 公分：BMI 16.5 至 32 為常備役體位，BMI 大於 32 至 35 為替代役體位，BMI 未滿 15 或超過 35 為免役體位。

試算結果僅供參考，實際結果仍以徵兵檢查及役政機關核定為準。規則來源：[內政部役政司體位區分標準](https://dca.moi.gov.tw/PhysicalStatus/)。

執行測試：`python -m unittest discover -s tests`

## 部署到 Azure Web App

`.github/workflows/deploy-azure.yml` 在 `main` 分支的應用程式、測試或 workflow 異動時執行，也可以從 GitHub Actions 手動觸發。流程會在 Windows runner 上執行測試、下載官方 Python 3.14.8 可嵌入版本，將 Python 與相依套件一起部署到 `NTUSTweb00`（資源群組 `NTUSTlab2026`）。站點透過 `web.config` 的 IIS HttpPlatformHandler 啟動 `serve.py`，以 Waitress 提供 Flask 服務。

此方法使用**隨部署包提供的 Python**，不依賴 Windows App Service 內建 Python 執行環境；HttpPlatformHandler 必須在目標站點可用。`NTUSTweb00` 目前是 Windows Web App，部署時會確認作業系統。

在 GitHub 儲存庫 **Settings → Secrets and variables → Actions** 確認：

- Repository secret `AZURE_CREDENTIALS`：Azure 服務主體的 `clientId`、`clientSecret`、`subscriptionId`、`tenantId` 組成的 JSON。此 Secret 已設定，不要把內容寫進程式碼。

請確認該服務主體對 `NTUSTweb00` 具有部署權限（例如該站點的 **Website Contributor**）。推送符合上述條件的 `main` 分支異動，或從 GitHub Actions 手動執行工作流程即可部署。
