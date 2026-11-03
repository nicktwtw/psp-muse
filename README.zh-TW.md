[English](README.md) | 繁體中文

# psp-muse

**讓 2008 年的 PSP-3000 重生，成為 Meta Muse 的語音終端。**

PSP-3000 上市十七年後，一台沒有藍牙、沒有觸控螢幕的粉紅色 PSP，也能跟 AI 助理開口對話。按住 R 鍵說話，Muse 就會出聲回答。這個專案使用 Muse Gadgets 裝置 SDK（hatch-link 藍牙配對協定），讓退役的硬體獲得第二人生，成為通往 Muse 的實體入口。

本專案為 **Meta 全球 AI 開發者黑客松**（Indie 個人賽道）參賽作品。

## 運作方式

- PSP 上執行原生自製程式（EBOOT），基礎是 [wobsoriano/muse-gadget-psp](https://github.com/wobsoriano/muse-gadget-psp)（Apache-2.0 授權），採用其 Rust 移植版。
- 一台具備藍牙低功耗（BLE）轉接器的 Linux 主機，負責與 Muse 進行一次性配對，並把裝置身分寫入 PSP 的記憶卡。
- 按住 **R** 鍵對麥克風說話：錄音會加密後送往 Meta Muse，回答再透過文字轉語音播放出來。
- 螢幕上有自製頭像，顯示裝置狀態（待機、聆聽、思考、說話）。

## 硬體

- Sony PSP-3000，韌體 6.61 加 ARK-4 自製韌體
- 具備 BLE 藍牙轉接器的 Linux 主機（負責配對與安裝）
- PSP 本身不需任何硬體改裝

## 自己動手做

以下全部依照上游專案
[wobsoriano/muse-gadget-psp](https://github.com/wobsoriano/muse-gadget-psp)
（Apache-2.0 授權）撰寫。你用自己的憑證從原始碼編譯，沒有任何預先
編譯好的檔案，也不涉及別人的金鑰或配對資料。

### 需要準備的東西

- 一台有內建麥克風的 PSP-3000（PSP-1000 與 PSP-2000 沒有內建麥克風；
  上游作者在 PSP-3001 上開發與測試）。
- 自製韌體：系統軟體 6.61 加 ARK。ARK 才能讓 PSP 加入 WPA2 Wi-Fi
  網路，原廠韌體做不到。
- 一張約有 5 MB 空間的記憶卡。
- 一個 Muse 帳號、手機上的 Muse App，以及從 gadgets.muse.ai 取得的
  SDK token。
- 一台用來編譯的電腦（上游作者用 macOS，Linux 理論上也可以）。
- 一台有藍牙的第二裝置，只用於一次性配對（樹莓派或跑 SDK Linux
  客戶端的 Linux 電腦）。
- 一把 OpenAI API 金鑰，用於語音。沒有金鑰的話，PSP 還是聽得到你說
  話、Muse 也還是會回答，只是 PSP 不會出聲。

### 第一步：取得工具

用 rustup 安裝 Rust，再安裝 PSP 編譯工具：

```sh
cargo install cargo-psp
```

專案在 `rust-toolchain.toml` 裡固定了 nightly 工具鏈，第一次編譯時
rustup 會自動取得。另外有一個步驟需要 C 編譯器。

### 第二步：產生頭像

App 播放的頭像是 Muse Gadget SDK 內附的頭像。那是 Meta 的東西，SDK
授權不包含它，所以它不在上游儲存庫裡。你要從自己的一份 SDK 複本產
生它：

```sh
git clone https://github.com/facebookincubator/muse-gadget-sdk vendor/muse-gadget-sdk
tools/avatar.sh
```

這一步會產生的檔案：

- `assets/avatar/avatar.bin`，每種心情各一段短動畫。

SDK 複本與產生的頭像檔都會被 git 忽略。

### 第三步：自己編譯 EBOOT

```sh
cargo psp --release
```

這一步會產生的檔案：

- `target/mipsel-sony-psp/release/EBOOT.PBP`

這就是 PSP 執行檔，由你在自己的電腦上產生，不提供預先編譯好的
EBOOT。

### 第四步：用藍牙跟 Muse 配對（只做一次）

Muse 是用藍牙跟裝置配對的，而 PSP 沒有藍牙。所以由一台有藍牙的裝
置先完成這段對話，產出的東西很小：一個身分與一對 token，再把這些
交給 PSP。之後 PSP 就自己連上 Muse，也會自己更新憑證。

使用 Muse Gadget SDK 的 Linux 客戶端：

1. 把它裝在樹莓派或有藍牙的 Linux 電腦上，照 SDK Linux 客戶端的
   README 從 Muse App 完成配對。
2. 把它停掉，讓它不再佔用這組配對：

```sh
sudo systemctl disable --now musegadget
```

3. 把 `identity.json` 與 `pairing.json` 從 `/var/lib/musegadget` 複
   製到你的電腦。
4. 把它們交給專案：

```sh
tools/pair.py path/to/the/folder
```

這一步會產生的東西：

- 儲存在 `state/` 裡的配對資料，git 會忽略它，請當成密碼看待。

有兩條規則很重要：同一個身分一次只能給一台裝置用，而且每次更新
憑證都會換掉 token。所以 PSP 拿到配對之後，配對來源那台客戶端要
保持關閉，或是直接移除。

### 第五步：複製到 PSP

把你的 SDK token 寫進專案資料夾裡名為 `.sdk_token` 的檔案，把
OpenAI 金鑰寫進 `.openai_key`。這兩個檔案都會被 git 忽略。

把 PSP 切到設定裡的 USB 連線，再執行：

```sh
tools/install.py
```

這一步會做的事：

- 把 App 與頭像複製到記憶卡上的 `PSP/GAME/Muse`，然後退出 USB。

每次重新編譯後都要再跑一次。它不會覆蓋記憶卡上已有的配對。

### 第六步：連接 Wi-Fi（這不是配對）

Wi-Fi 設定跟藍牙配對是兩回事。啟動自製韌體後，先在 PSP 的網路設
定裡連一次 Wi-Fi，確認日期時間正確，再從遊戲選單執行 Muse。

App 會連上網路設定裡儲存的第一組連線。啟動時按住 L，就會改連第
二組。

### 第七步：開機並測試連線

螢幕角落會顯示 PSP 能不能連上 Muse，頭像則顯示它正在做什麼：

- 琥珀色燈號與 connecting：正在加入 Wi-Fi 或連接 Muse。
- 綠色燈號與 online：已就緒，按住 R 就可以說話。
- 紅色燈號與 offline：Muse 已經 45 秒連不上，或 PSP 尚未配對，App
  會繼續嘗試。
- 頭像藍色：按住 R 聆聽中。
- 頭像粉紅色加思考點點：問題正在送出，或 Muse 正在回答。
- 頭像綠色加音波：正在說出答案。
- 只有幾秒愛心、沒有聲音：Muse 已經回答，但 PSP 無法說出來（例
  如沒有提供語音金鑰）。

測試方式：按住 R 說話，放開送出。按 HOME 離開。

如果運作不正常，先讀記憶卡上 App 旁邊的 `log.txt`（前一次的紀錄
保留為 `log-before.txt`）。紀錄裡會寫 Wi-Fi 網路名稱與訊號強度，
這是最先要檢查的。注意 PSP 的無線電比較弱，需要夠強的 Wi-Fi 訊
號，而且太長的回答只會說出前面約 420 個字元。

你也可以從任何裝置叫 Muse 對 PSP 下指令，記得指名裝置：`psp.say`
會讓 PSP 說出訊息，`psp.dance` 會讓頭像跳舞，`psp.status` 會讀取
電池與 Wi-Fi 狀態。

## 目前進度

- [x] EBOOT 編譯完成，並能在實機啟動（Rust 移植版）
- [x] 自製頭像已產生並能顯示
- [x] SDK token 已取得；一次性藍牙配對完成
- [x] 把裝置身分安裝到記憶卡，並確認上線狀態
- [x] 在 PSP 上完成第一次端到端語音對話
- [x] 示範影片（70 秒，已檢查）

## 安全說明

identity.json、pairing.json 與 SDK token 是每台裝置各自的機密資料，已列入 .gitignore，永遠不會提交到這個儲存庫。

## 致謝與授權

本專案以 wobsoriano 的開源專案 muse-gadget-psp 為基礎（Apache License 2.0），詳見上游專案的 LICENSE 與 NOTICE。
