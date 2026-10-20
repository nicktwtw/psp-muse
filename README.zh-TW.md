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

## 目前進度

- [x] EBOOT 編譯完成，並能在實機啟動（Rust 移植版）
- [x] 自製頭像已產生並能顯示
- [x] SDK token 已取得；一次性藍牙配對完成
- [ ] 把裝置身分安裝到記憶卡，並確認上線狀態
- [ ] 在 PSP 上完成第一次端到端語音對話
- [ ] 示範影片

## 安全說明

identity.json、pairing.json 與 SDK token 是每台裝置各自的機密資料，已列入 .gitignore，永遠不會提交到這個儲存庫。

## 致謝與授權

本專案以 wobsoriano 的開源專案 muse-gadget-psp 為基礎（Apache License 2.0），詳見上游專案的 LICENSE 與 NOTICE。
