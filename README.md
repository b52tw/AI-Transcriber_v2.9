# Junba AI Transcriber v2.9

Windows 10/11 x64｜繁體中文 GUI｜離線 Whisper + Intel NPU/GPU + NVIDIA CUDA + Google Gemini。

## v2.9 Intel NPU 執行期備援

部分 Intel Core Ultra（含 Arrow Lake NPU 3720）在 OpenVINO Whisper 已成功編譯後，仍可能於第一次 `generate()` 回傳 Level Zero `ZE_RESULT_ERROR_INVALID_ARGUMENT (0x78000004)`。v2.9 不再終止工作：會先重建 NPU 相容模式，再自動切到 Intel GPU，最後可退回 OpenVINO CPU；同一區段直接續跑。自動效能模式仍優先使用 Intel GPU 而不是 NPU。

新增 `large-v3-turbo`，適合希望比 large-v3 更快、又維持多語辨識能力的情境。

## v2.9 主要更新

- 新增「硬體加速」選單：
  - 自動（效能優先）
  - NVIDIA GPU / CUDA
  - Intel GPU / OpenVINO
  - Intel NPU / OpenVINO（省電）
  - CPU / faster-whisper（Intel/AMD）
- 啟動/重新偵測時會列出 CPU、CUDA、OpenVINO GPU/NPU 與實際裝置名稱。
- Intel NPU/GPU 使用 **OpenVINO GenAI WhisperPipeline**。
- OpenVINO 模型採官方 `OpenVINO/whisper-*-int8-ov`：large-v3 / medium / small / base。
- OpenVINO 模型第一次使用自動下載至使用者資料夾，之後可離線重用。
- CPU 路徑繼續使用 faster-whisper/CTranslate2 INT8；Intel 與 AMD x86-64 CPU 都可用。
- NVIDIA GPU 繼續使用 faster-whisper/CTranslate2 CUDA + FP16。
- 保留 v2.5 的「立即停止並輸出目前結果」、切割/合併、Gemini、多講者、繁體中文、Word/TXT/SRT/VTT。

## 你這台 Intel AI Boost / Intel Arc 電腦

如果 OpenVINO 與 Intel 驅動正常，程式應同時看到：

- `Intel NPU / OpenVINO`（工作管理員 NPU 0 / Intel(R) AI Boost）
- `Intel GPU / OpenVINO`（Intel Arc）
- `CPU / faster-whisper（Intel）`

若要明確讓工作管理員的 NPU 使用率上升，請不要選「自動」，而是直接選：

**Intel NPU / OpenVINO（省電）**

「自動（效能優先）」的順序為：NVIDIA CUDA → Intel GPU → Intel NPU → CPU。

> 16 GB RAM 的電腦如果當下記憶體已高占用，large-v3 第一次 NPU/GPU 編譯可能較吃記憶體。可先關閉大型程式，或改用 medium / small。

## AMD / Intel CPU 相容性

CPU 模式使用 CTranslate2 預編譯 x86-64 路徑。CTranslate2 會依 CPU 自動選擇適合的指令集；Intel 與 AMD CPU 都可使用。AMD Radeon GPU **本版尚未做 GPU 加速**，AMD 電腦會使用 CPU 模式，除非另有 NVIDIA CUDA GPU。

## Intel NPU / GPU 模型

OpenVINO 模式會自動下載對應模型：

- large-v3 → `OpenVINO/whisper-large-v3-int8-ov`
- medium → `OpenVINO/whisper-medium-int8-ov`
- small → `OpenVINO/whisper-small-int8-ov`
- base → `OpenVINO/whisper-base-int8-ov`

第一次下載需要網路；完成後模型留在本機，之後可以離線辨識。

## 音檔流程

1. 加入或拖曳音檔。
2. 選擇是否先切割（2/5/10/15/30/60 分鐘或自訂）。
3. 選辨識引擎：離線 Whisper / Google Gemini / 混合模式。
4. 若使用離線 Whisper，選硬體加速。
5. 開始處理；切割會先完整完成，再開始辨識。
6. 可暫停、繼續、或「立即停止並輸出目前結果」。
7. 匯出 Word/TXT/SRT/VTT。

## Google Gemini

- 支援音訊上傳轉錄。
- 多人講者 + 字詞時間戳可同時使用；Smart 智慧逐字稿與這兩項互斥，GUI 會自動切換。
- 中文/華台混合輸出可自動轉為台灣繁體中文。
- API Key 儲存在 Windows 認證儲存區，不寫死在 EXE。

## GitHub Actions 建置

`.github/workflows/build-windows-v2.9.yml` 會在 `windows-latest`：

- 安裝 PySide6 / faster-whisper / OpenVINO / OpenVINO GenAI 等依賴
- compileall
- pytest
- 固定 OpenVINO ABI 三套版本（openvino / openvino-tokenizers / openvino-genai）
- 在 Windows 實際執行 `Core.add_extension(openvino_tokenizers.dll)`，不是只測 import
- OpenVINO import 與裝置列舉 smoke test
- Source GUI offscreen self-test
- 建置 Single EXE
- Single EXE self-test + UI self-test
- 建置 Portable 版
- Portable EXE self-test + UI self-test

成功後會有兩個 Artifact：

- `Junba-AI-Transcriber-v2.9-Single-EXE-Windows-x64`
- `Junba-AI-Transcriber-v2.9-Portable-Windows-x64`

Intel NPU/GPU 請**優先使用 Portable 版**；它保留 OpenVINO DLL 的原始目錄結構，最穩定。Single EXE 只有在 GitHub Windows runner 通過 `openvino_tokenizers.dll` 實際載入測試後才會上傳。

## 注意

- NPU 需要 Windows 11 與正確的 Intel NPU 驅動；程式會以 OpenVINO 實際列出的裝置為準。
- Intel GPU 需要正確的 Intel Graphics Driver。
- 模型沒有內嵌進 EXE，所以第一次使用新的 Whisper/OpenVINO 模型需要下載。


## v2.9 重要修正
- 修正 Windows GUI EXE 第一次下載 OpenVINO/Hugging Face 模型時 `NoneType.write`。
- Hugging Face console progress bar 關閉，改以 GUI 顯示下載等待狀態。
- FFmpeg/PowerShell 等背景工具使用 Windows `CREATE_NO_WINDOW`，避免跳出大量子視窗。
- Intel NPU 編譯若第一次失敗，會自動嘗試 Intel 相容模式一次，再提供明確錯誤。
- 修正 v2.7 `openvino_tokenizers.dll` WinError 126/127：固定 `openvino==2026.4.0`、`openvino-tokenizers==2026.4.0.0`、`openvino-genai==2026.4.0.0`，並將 Tokenizers/GenAI DLL 明確收進 PyInstaller。
- 啟動 OpenVINO 前自動加入 DLL 搜尋路徑；錯誤訊息會區分「DLL/ABI 問題」與「NPU/GPU 編譯問題」，不再把所有錯誤都誤判成 NPU Driver。
