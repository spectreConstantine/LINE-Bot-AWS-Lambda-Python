# 使用 AWS Lambda 快速部署 LINE Bot
( 搭配 AWS API Gateway [並隨附範例 LineBot_AWS_Lambda_QuickEx.zip](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/raw/main/LineBot_AWS_Lambda_QuickEx.zip "範例LineBot_AWS_Lambda_QuickEx.zip") )
-[本文 PowerPoint(pdf) 版](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/raw/main/%E4%BD%BF%E7%94%A8%20AWS%20Lambda%20%E9%83%A8%E7%BD%B2%20LINE%20Bot.pdf) 

## 什麼是 AWS Lambda? 用它來部署 Line Bot 有什麼不同和好處?

### 讓你專注在應用程式開發，不用架設伺服器、設定監控、網路安全性及建設基礎架構

- 它就像是 Python 中的一個 Lambda Function 。這是 Lambda 這個服務的簡單概念。
    - 應用程式即是一個 Function，**預設是 3 秒鐘**，最長可執行 15 分鐘。
    - 應用程式執行的前與後，服務為停止狀態。
- **把應用程式寫成可執行的** Function **，其他由雲服務代管。這是** Function As a Service **的簡單概念。**
    - 雲服務提供Function的執行環境(Runtime)，執行你包裝好的Function 應用程式。
    - 應用程式的Function 只存在執行階段(Runtime)，執行環境終止也代表所有資料不會留存。
- **不用執行或架設任何伺服器，應用程式只會使用執行階段的運算時間。這是** Serverless **的簡單概念。**
    - 不用擔心或管理任何伺服器。由雲服務負責所有的其他工作；除了你開發的應用程式。
    - 使用雲服務提供的Free-Tier 方案。在方案提供的限額內永久免費。

## 用 Lambda 來部署 Line Bot 我需要具備什麼條件?

### 註冊的 LINE 與 AWS 服務帳號，以及與你要開發的應用程式相關的知識及其商業邏輯

- 註冊的 AWS 雲服務帳號
    - 可以註冊AWS Free-Tier 方案: https://aws.amazon.com/tw/free/ ，提供一年有限度的免費服務。
    - 註冊帳號需要有可收到信的email 帳號。需要提供有效的信用卡卡號(驗證USD$1 但不會收費)。
    - 註冊帳號過程為全自動化，過程中請確認輸入正確的資料，註冊完後服務即可使用。
- **註冊的** LINE **服務帳號**
    - 在LINE 的開發網站建立一個LINE Provider 和Message API Channel。
    - 啟用LINE Message API的webhook。稍後會說明細節。
- **對** Python **程式語言或其他** AWS Lambda **支援的程式語言有基本的認識與瞭解**
    - 參考雲服務商及LINE 服務商提供的SDK範例，協助你開發LINE Bot 應用程式。

## 有什麼注意事項?

### 注意安全性的控管，並關注 AWS Free-Tier 的免費限額!

- 安全性
    - 雲服務廠商專注於基礎架構及服務本身的安全性，使用者需自行控管應用程式本身的安全性。
    - 你使用的email 帳號與密碼及註冊帳號的密碼等安全性保護也很重要。
    - 如果疏於管理安全性，可能會導致服務被惡意使用造成**鉅額的使用費用**!
- **查看** AWS Free-Tier **的細節**
    - AWS 的Free-Tier 免費方案，提供 AWS Lambda 每月 **1 百萬個**免費請求(request)。
    - 如果需搭配使用其他雲服務儲存AWS Lambda 執行運算完的資料，需注意相關的費用與成本。
- **設定** AWS **費用的郵件提醒通知並定期查看** AWS **管理頁面的費用細節**
    - 即時收到費用通知，瞭解使用狀況。並在需要時與雲服務支援人員保持連繫以瞭解細節。

## 架構簡圖概觀

-   ![架構圖概觀](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114621.png)

## 步驟 1. 登入 AWS 管理界面建立 Lambda Function

### 預設的帳號為 Root Account 有最高的權限。可參考 AWS 文件啟用雙階段認證!

- 使用瀏覽器開啟 https://console.aws.amazon.com/
    - 使用註冊的email 帳號和密碼登入，或使用另外建立的 Administrative Access 的 IAM 帳號登入。細節請見後面**其他參考資訊**。
    - 登入後可以在右上角先選擇區域。可以使用 東京(ap-northeast-1) 或 香港(ap-east-1) 離台灣較近。
    - 需注意 香港(ap-east-1) 區域需在登入後額外啟用該區域。請參考AWS 網站的說明。
- 點取左上角的 **Services**，點選運算類別的 **Lambda** 服務開啟 **Lambda** 服務界面。按一下建立函式
    - 選最左邊的從頭開始撰寫，為函式命名。選Python 3.7 為執行時間，其他預設值不變，選 **建立函式** 。

    ![步驟1](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114626.png)
    
- 在新增觸發條件選按一下，選 **API Gateway**，**Create API**，**HTTP API**，安全性為開啟。按一下新增。

    ![步驟1](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114629.png)
    
    - 點選 API Gateway 下方詳細資訊，將 API 終端節點旁的網址填入 LINE Bot 的Webhook(參考步驟3)

    ![步驟1](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114632.png)   
    
## 步驟 2. 打包應用程式及相依套件上傳到 AWS Lambda

### 為簡化細節，此文件提供基礎的範例套件。僅需上傳一次。爾後直接在界面編寫程式

- 點畫面中間的 Lambda 圖示，在右側的操作上點一下選取上傳 .zip 檔案。
    - 預設的函式左邊只有一個根目錄和一個 .py 檔案。點選 .py 檔案後右側即是該檔案的程式碼內容。
    - 按上傳後選取隨附的 LINE Bot 及相關套件[範例LineBot_AWS_Lambda_QuickEx.zip](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/raw/main/LineBot_AWS_Lambda_QuickEx.zip "範例LineBot_AWS_Lambda_QuickEx.zip")，然後按儲存。

    ![步驟2](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114635.png) 
    
    - 上傳後的程式及套件會覆蓋原有的預設執行環境。如果已先在剛才的環境撰寫程式需先匯出備份。
    - 如果上傳超過 3MB 的.zip 會有訊息提醒你 Lambda 編輯畫面無法使用。需使用 AWS CLI 等工具。
- 將 **LINE Bot** 的 **Channel Access Token** 及 **Channel Secret** 新增到程式碼下方的環境變數 ( **參考步驟** 3)
    - 由於安全性的考量，這裡採用環境變數的方式儲存這兩個設定，再從程式碼裡用os.getenv去存取。

    ![步驟2](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114637.png)
    
    - 環境變數預設是自動使用 AWS KMS 加密的。在免費方案中採用 KMS 有提供有限的免費用量。
- 可以直接線上在中間畫面中修改你的 **LINE Bot** 程式。修改後按一下 **Deploy** 即可部署最後修訂的內容。

    ![其他](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114645.png) 
    
    - 在 AWS Lambda 裡也有提供發行新版本或匯出函式等多樣性的功能。請參考 AWS 網站取更多資訊。

## 步驟 3. 修改 LINE Message API Webhook 指向 AWS API Gateway

### 你的 Line Bot 應用程式已部署完成。讓 LINE Server 指向你的 Line Bot 產生關聯

- 開啟 LINE 的開發頁面 https://developers.line.biz/console/ 並登入你的 LINE 帳號建立 Provider
    - 建立Provider為建立Line Bot 的第一個步驟。建立完後選擇該Provider後點選建立Channel。
- **在建立** Channel **裡選擇建立** Message API **，並啟用** Usewebhook
    - 建立 Message API 的後，點選 Basic Settings 設定。在頁面中找到 Channel secret 的值。
    - 在下一個 Messaging API 頁籤中找到 Channel access token，點Issue，取得 token 的值。
    - 回到 Basic Settings 按一下LINE Official Account Manager 的連結，選取左邊 Message API。
    - 在 Webhook 網址中填入步驟 1 中的 API Gateway 下方詳細資料的 API 終端節點。按儲存。

    ![步驟3](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114640.png) 
    
    - 回LINE Developers 頁面Messaging API 頁籤，找到Webhook settings，將Use webhook 啟用。

    ![步驟3](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114643.png) 
    
- **完成啦** ~ **這樣你的** LINE Bot **就可以使用囉**! **是不是很容易呢**?
    - 在 Messaging API 頁籤中找 Bot information ，打開手機的LINE 掃描 QR Code 加入好友即可對話。可以參考文末擷圖。

## 其他參考資訊

#### 如何打包套件?

- 建立一個資料夾。使用 pip install **Package** -t .。**然後進入資料夾內**，全選，右鍵，傳送到，壓縮的資料夾。
- https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/python-package.html
#### 開發 **LINE Bot** 時碰到錯誤或問題如何除錯 (Debug/Troubleshooting)?
- 所有的記錄包含Print() 函式的內容會包含在CloudWatch的Log Group 裡。請參考後面的螢幕擷圖。

    ![其他](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_114648.png) 
    
#### 建立您的第一個 IAM **管理員用戶和群組**
- https://docs.aws.amazon.com/zh_tw/IAM/latest/UserGuide/getting-started_create-admin-group.html
#### AWS Lambda **常見問答集**
- https://aws.amazon.com/tw/lambda/faqs/
#### LINEBot SDK for Python
- https://github.com/line/line-bot-sdk-python
#### AWS Lambda Deployment Package in Python
- https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/python-package.html
#### AWS Lambda Runtimes
- https://docs.aws.amazon.com/zh_tw/lambda/latest/dg/lambda-runtimes.html
#### LINE Flex Message Simulator
- https://developers.line.biz/flex-simulator/
#### JSON Formatter & Validator
- https://jsonformatter.curiousconcept.com/
#### Lambda Deployment Packages with Docker
- https://blog.quiltdata.com/an-easier-way-to-build-lambda-deployment-packages-with-docker-instead-of-ec2-9050cd486ba8
#### 歡迎試用一下我用相同的方法建立的 桃園市圖書館 Line Bot: @082gynfz

- 除了使用上述的 AWS Lambda/AWS API Gateway, 也搭配了 AWS S3/DynamoDb/SQS 及 EC2/MongoDB 等服務。
    ![桃園市圖書館](https://github.com/spectreConstantine/LINE-Bot-AWS-Lambda-Python/blob/main/2020-10-04_130228.png) 




```
文章by alvin.constantine@outlook.com © 2020 All rights reserved
```
