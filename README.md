# temper-note
1. 目的（Purpose）
解決單音「基本頻率估計」（monophonic pitch tracking / F0 estimation）
取代與改進傳統 DSP + heuristics（例如：pYIN、SWIPE）
提供更穩健噪聲和泛化能力（跨音源與演唱/說話）
可以直接用於 MIR、語音處理、音樂分析、音高標註等
2. 創新點（Novelty）
時域端到端 CNN：
直接用原始 waveform 取代先 FFT / CQT 等時頻特徵（差異化）
對音高做 360-bin 連續化 Soft-label 表示：
每 10ms 一點，對應 360 個 pitch bins（20 cents 分辨）
直接對 F0 做分類 + 回歸：
不用傳統周期性/傅立葉 peak 找最大值
訓練資料混合：
錄音（MIR-1K、MedleyDB、Bach10、等） + 合成（RWC-Synth、NSynth）
泛化與噪聲測試：
在不同 SNR 和不同音源上比較
開源釋出整套工具：
marl/crepe PyPI 包 + 预训练模型（實用性高）
3. 方法（Method）
輸入
16 kHz 單聲道 waveform (0.1024s frame, 1ms shift ？論文 10ms hop)
模型
6 層 1D convolution + 2 層全連接 (FC)
輸出 360 bins (pitch space 6.875 Hz–x)
sigmoid / softmax-like activation
預測
每 10ms 產生 360 維 “activation”
argmax + 局部加權平均取 pitch：
類似論文 / 現實框架（GitHub 版有 argmax_local_weighted_average）
監督標籤
1-hot 的“高斯分布 smooth label”中心在真實 F0
損失
binary cross-entropy 或 categorical CE（paper 可能 BCE）
訓練
epoch 多、batch 小、資料增强 (noise)
augmentation ＝ 各種 SNR 噪音加在原聲上
後處理
voicing threshold 由 activation energy 決定
optional Viterbi smoothing
4. 實驗（Experiments / Evaluation）
比較對象
pYIN（基準）, SWIPE、YPinyin, 家算法
共用數據集
MIR-1K, MedleyDB, Bach10, RWC-Synth, MDB-STEM-Synth, NSynth
情景
Clean, +SNR 0 / 10 / 20 / 30, noise types
voice + instrument（歌 / 語 / 樂器）混合
指標
Raw pitch accuracy (RPA, cents error), Raw chroma accuracy (RCA)
voicing precision / recall / F1
robustness across speakers、音高範圍
額外
inference speed
model capacity tradeoff（tiny/small/medium/large/full）
5. 結果（Results）
CREPE 在 RPA/RCA 上超越或持平 pYIN
噪聲場景
CREPE 在低 SNR 下仍領先（> pYIN/SWIPE）
資料集泛化
未見數據測試仍好
單聲源音色差異
human voice / instrument 在模型中均表現一致
結論：采用深度 CNN 的時域學習可達到“實用準確度 + 噪聲韌性”
兼容：
後續很多解法（Vocal melody extraction, source separation的 F0 module）引用並改寫


<img width="8192" height="746" alt="Monophonic Pitch Detection-2026-03-29-111011" src="https://github.com/user-attachments/assets/09c5bd9f-cd7c-4a72-9ddf-dd703e0d3803" />
