# WebFEA - 二維框架分析

**[English](./README.md) | [繁體中文](#)**

---

## 概述

**WebFEA** 是一款全面的網頁結構工程應用程式，用於分析和設計二維框架結構和桁架系統。該工具提供即時有限元分析、互動式畫布視覺化，以及支援複雜載重條件的自動結構計算，包括分佈載重、點載重、支承沉陷，以及具有局部/全域座標系統的構件特定載重。

---

## 核心功能

### 計算引擎

* **直接剛度法**：經典矩陣結構分析，每節點 3 個自由度 (Δx, Δy, θz)
* **構件類型**：
  - 框架構件（含彎曲、軸向和剪力剛度）
  - 桁架構件（鉸接，僅軸力）
* **載重類型**：
  - 節點點載重 (Fx, Fy, M)
  - 支承沉陷（規定位移）
  - 構件點載重（跨距上任意位置）
  - 分佈載重（均佈或梯形，部分或全跨）
  - 載重角度參考：全域或局部座標系統
* **支承條件**：固定端、鉸支承和滾支承，可旋轉角度
* **結果**：
  - 節點位移 (Δx, Δy, θz)
  - 支承反力 (Rx, Ry, Rm)
  - 構件內力（軸力 N、剪力 V、彎矩 M）
  - 變形形狀視覺化
  - 內力圖（BMD、SFD、AFD）

### 使用者介面

* **雙語支援**：英文與繁體中文即時切換
* **即時視覺化**：
  - 互動式畫布，支援平移和縮放
  - 動態網格系統（二進位/十進位細分）
  - 結構上疊加即時力圖
  - 誇大比例的變形形狀
  - 懸停工具提示顯示任意點的內力
* **互動工具**：
  - 節點放置（網格對齊）
  - 構件建立（點擊起點-終點節點）
  - 支承指定（角度控制）
  - 載重施加（視覺回饋）
  - 選擇模式（編輯/刪除）
* **斷面庫**：
  - 預定義標準斷面（混凝土、鋼 I 型梁、管）
  - 自訂斷面計算器（7 種形狀類型）
  - 自動屬性計算（面積、慣性矩）
  - 材料資料庫（彈性模數 E）
* **進階功能**：
  - 復原/重做功能 (Ctrl+Z, Ctrl+Y)
  - 自動儲存至瀏覽器本地儲存
  - 專案檔案匯出/匯入（JSON 格式）
  - 範例模型（門架、桁架橋）
  - 深色/淺色主題切換
  - 顯示選項面板（圖表、反力、比例）

---

## 結構分析理論

### 1. 座標系統

#### 全域座標系統
- **X 軸**：水平，向右為正
- **Y 軸**：垂直，向上為正
- **原點**：使用者定義（通常在畫布左下角）

#### 局部構件座標系統
- **x 軸**：沿構件從節點 i 到節點 j
- **y 軸**：垂直於構件（右手法則）
- **轉換角度** α = atan2(Δy, Δx)

---

### 2. 構件剛度矩陣

#### 2.1 框架構件（6×6 局部剛度）

對於具有以下特性的框架構件：
- 長度：L (m)
- 彈性模數：E (kPa)
- 斷面積：A (m²)
- 慣性矩：I (m⁴)

**局部自由度排序**：[u₁, v₁, θ₁, u₂, v₂, θ₂]

**局部剛度矩陣**：

$$
\mathbf{k}_{\text{local}} = \begin{bmatrix}
k_1 & 0 & 0 & -k_1 & 0 & 0 \\
0 & k_2 & k_3 & 0 & -k_2 & k_3 \\
0 & k_3 & k_4 & 0 & -k_3 & k_5 \\
-k_1 & 0 & 0 & k_1 & 0 & 0 \\
0 & -k_2 & -k_3 & 0 & k_2 & -k_3 \\
0 & k_3 & k_5 & 0 & -k_3 & k_4
\end{bmatrix}
$$

其中：

$$
k_1 = \frac{EA}{L}
$$

$$
k_2 = \frac{12EI}{L^3}
$$

$$
k_3 = \frac{6EI}{L^2}
$$

$$
k_4 = \frac{4EI}{L}
$$

$$
k_5 = \frac{2EI}{L}
$$

**物理意義**：
- k₁：軸向剛度
- k₂：橫向剪力剛度
- k₃：剪力-彎矩耦合
- k₄：近端旋轉剛度
- k₅：遠端旋轉剛度（由於對稱性為 k₄ 的一半）

#### 2.2 桁架構件（6×6 局部剛度）

對於桁架構件，彎曲剛度為零：

$$
k_2 = k_3 = k_4 = k_5 = 0
$$

**奇異性修補**：防止旋轉自由度對角線為零：

$$
k_{\text{local}}[2,2] = k_{\text{local}}[5,5] = 1.0 \text{ (若 } |k_4| < 10^{-9}\text{)}
$$

#### 2.3 轉換矩陣（6×6）

**方向餘弦**：

$$
c = \frac{\Delta x}{L} = \cos \alpha
$$

$$
s = \frac{\Delta y}{L} = \sin \alpha
$$

**轉換矩陣**：

$$
\mathbf{T} = \begin{bmatrix}
c & s & 0 & 0 & 0 & 0 \\
-s & c & 0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & c & s & 0 \\
0 & 0 & 0 & -s & c & 0 \\
0 & 0 & 0 & 0 & 0 & 1
\end{bmatrix}
$$

#### 2.4 全域剛度矩陣

$$
\mathbf{k}_{\text{global}} = \mathbf{T}^T \mathbf{k}_{\text{local}} \mathbf{T}
$$

**組裝**：每個構件的全域剛度組裝到結構剛度矩陣 **K** 的自由度位置：

- 節點 i：[3i, 3i+1, 3i+2]
- 節點 j：[3j, 3j+1, 3j+2]

---

### 3. 載重向量組裝

#### 3.1 節點點載重

施加的力直接加到全域力向量 **F**：

$$
F[3n] \mathrel{+}= F_x
$$

$$
F[3n+1] \mathrel{+}= F_y
$$

$$
F[3n+2] \mathrel{+}= M_z
$$

其中 n 為節點索引。

#### 3.2 構件載重（固端作用力）

構件載重（分佈或跨上點載重）使用**固端作用力**公式轉換為等效節點力，然後施加到全域力向量。

##### 3.2.1 跨上點載重

**輸入參數**：
- 大小：P (kN)
- 角度：θ（度，從水平方向測量）
- 距節點 i 距離：a (m)
- 構件長度：L (m)
- 角度參考："global" 或 "local"

**步驟 1：確定全域角度**

若 angleRef = "local"：
$$
\theta_{\text{global}} = \theta_{\text{beam}} + \theta_{\text{input}}
$$

其中：
$$
\theta_{\text{beam}} = \arctan\left(\frac{\Delta y}{\Delta x}\right)
$$

若 angleRef = "global"：
$$
\theta_{\text{global}} = \theta_{\text{input}}
$$

**步驟 2：分解為全域分量**

$$
F_x^{\text{global}} = P \cos(\theta_{\text{global}})
$$

$$
F_y^{\text{global}} = P \sin(\theta_{\text{global}})
$$

**步驟 3：轉換到局部梁座標**

$$
P_x = F_x^{\text{global}} \cos(\theta_{\text{beam}}) + F_y^{\text{global}} \sin(\theta_{\text{beam}})
$$

$$
P_y = -F_x^{\text{global}} \sin(\theta_{\text{beam}}) + F_y^{\text{global}} \cos(\theta_{\text{beam}})
$$

**步驟 4：計算距遠端的距離**

$$
b = L - a
$$

**步驟 5：固端作用力（橫向分量 P_y）**

**彎矩**（符號慣例：逆時針為正）：

$$
M_1 = \frac{P_y \cdot a \cdot b^2}{L^2}
$$

$$
M_2 = -\frac{P_y \cdot a^2 \cdot b}{L^2}
$$

**剪力**：

$$
V_1 = \frac{P_y \cdot b^2 (3a + b)}{L^3}
$$

$$
V_2 = \frac{P_y \cdot a^2 (a + 3b)}{L^3}
$$

**軸力**（線性分佈）：

$$
N_1 = -P_x \cdot \frac{b}{L}
$$

$$
N_2 = -P_x \cdot \frac{a}{L}
$$

**步驟 6：組裝局部固端力向量**

$$
\mathbf{f}_{\text{FE,local}} = \begin{bmatrix} N_1 \\ V_1 \\ M_1 \\ N_2 \\ V_2 \\ M_2 \end{bmatrix}
$$

**步驟 7：轉換到全域**

$$
\mathbf{f}_{\text{FE,global}} = \mathbf{T}^T \mathbf{f}_{\text{FE,local}}
$$

**步驟 8：施加到全域力向量**

$$
F[3i] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[0]
$$

$$
F[3i+1] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[1]
$$

$$
F[3i+2] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[2]
$$

$$
F[3j] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[3]
$$

$$
F[3j+1] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[4]
$$

$$
F[3j+2] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[5]
$$

##### 3.2.2 分佈載重（梯形）

**輸入參數**：
- 起始大小：w₁ (kN/m)
- 結束大小：w₂ (kN/m)
- 起始距離：d₁ (m)
- 結束距離：d₂ (m)
- 角度：θ（度）
- 角度參考："global" 或 "local"

**積分方法**：使用**辛普森法則**進行數值積分（10 步）

**步驟 1：確定全域角度**（與點載重相同）

**步驟 2：計算步長**

$$
\Delta L = d_2 - d_1
$$

$$
h = \frac{\Delta L}{n} \quad (n = 10 \text{ 步})
$$

**步驟 3：循環積分點**

對於 i = 0 到 n：

$$
x = d_1 + i \cdot h
$$

**插值參數**：

$$
s = \frac{x - d_1}{\Delta L}
$$

**在 x 處的載重強度**：

$$
w_y(x) = w_1 (1 - s) + w_2 \cdot s
$$

$$
w_x(x) = w_1 (1 - s) + w_2 \cdot s
$$

分解角度後：

$$
n_x = \cos(\theta_{\text{global}}), \quad n_y = \sin(\theta_{\text{global}})
$$

$$
w_x(x) = w(x) \cdot (n_x \cos\theta_{\text{beam}} + n_y \sin\theta_{\text{beam}})
$$

$$
w_y(x) = w(x) \cdot (-n_x \sin\theta_{\text{beam}} + n_y \cos\theta_{\text{beam}})
$$

**辛普森權重**：

$$
\text{weight} = \begin{cases}
1 & \text{若 } i = 0 \text{ 或 } i = n \\
4 & \text{若 } i \text{ 為奇數} \\
2 & \text{若 } i \text{ 為偶數}
\end{cases}
$$

$$
w_{\text{eff}} = \frac{\text{weight} \cdot h}{3}
$$

**微分固端作用力**（將 dw 視為點載重）：

距遠端距離：

$$
b = L - x
$$

**微分彎矩**：

$$
dM_1 = \frac{w_y(x) \cdot x \cdot b^2}{L^2} \cdot w_{\text{eff}}
$$

$$
dM_2 = -\frac{w_y(x) \cdot x^2 \cdot b}{L^2} \cdot w_{\text{eff}}
$$

**微分剪力**：

$$
dV_1 = \frac{w_y(x) \cdot b^2 (3x + b)}{L^3} \cdot w_{\text{eff}}
$$

$$
dV_2 = \frac{w_y(x) \cdot x^2 (x + 3b)}{L^3} \cdot w_{\text{eff}}
$$

**微分軸力**：

$$
dN_1 = -w_x(x) \cdot \frac{b}{L} \cdot w_{\text{eff}}
$$

$$
dN_2 = -w_x(x) \cdot \frac{x}{L} \cdot w_{\text{eff}}
$$

**累積**：

$$
N_1 \mathrel{+}= dN_1, \quad V_1 \mathrel{+}= dV_1, \quad M_1 \mathrel{+}= dM_1
$$

$$
N_2 \mathrel{+}= dN_2, \quad V_2 \mathrel{+}= dV_2, \quad M_2 \mathrel{+}= dM_2
$$

**轉換和施加**（與點載重的步驟 6-8 相同）

#### 3.3 支承沉陷

對於支承節點的規定位移 (Δx, Δy, θ)：

**加到已知位移向量**：

$$
D_{\text{known}}[3n] = \Delta x
$$

$$
D_{\text{known}}[3n+1] = \Delta y
$$

$$
D_{\text{known}}[3n+2] = \theta
$$

**標記自由度為受約束**：

$$
\text{restrainedDofSet.add}(3n), \quad \text{restrainedDofSet.add}(3n+1), \quad \text{restrainedDofSet.add}(3n+2)
$$

---

### 4. 方程組系統

#### 4.1 分割

**自由自由度**：無支承約束的自由度

**受約束自由度**：有支承或規定位移的自由度

**分割全域剛度**：

$$
\begin{bmatrix}
\mathbf{K}_{ff} & \mathbf{K}_{fr} \\
\mathbf{K}_{rf} & \mathbf{K}_{rr}
\end{bmatrix}
\begin{bmatrix}
\mathbf{D}_f \\
\mathbf{D}_r
\end{bmatrix}
=
\begin{bmatrix}
\mathbf{F}_f \\
\mathbf{F}_r
\end{bmatrix}
$$

#### 4.2 有效力向量

考慮規定位移：

$$
\mathbf{F}_{f,\text{eff}} = \mathbf{F}_f - \mathbf{K}_{fr} \mathbf{D}_r
$$

#### 4.3 自由自由度的解

$$
\mathbf{K}_{ff} \mathbf{D}_f = \mathbf{F}_{f,\text{eff}}
$$

**求解器**：高斯消去法配合部分樞軸選取

**前向消去**：

對於 i = 0 到 n-1：

1. 找到樞軸行（k ≥ i 時 |K[k][i]| 的最大值）
2. 交換第 i 行和 maxRow 行
3. 檢查奇異性：|K[i][i]| < 10⁻¹⁰ → 錯誤
4. 消去樞軸下方的列：

$$
\text{factor} = \frac{K[k][i]}{K[i][i]}
$$

$$
K[k][j] \mathrel{-}= \text{factor} \cdot K[i][j] \quad \forall j \geq i
$$

**回代**：

對於 i = n-1 到 0：

$$
D_f[i] = \frac{F_{f,\text{eff}}[i] - \sum_{j=i+1}^{n-1} K[i][j] D_f[j]}{K[i][i]}
$$

#### 4.4 完整位移向量

$$
\mathbf{D} = \begin{cases}
D_f[k] & \text{若自由度為自由} \\
D_r[k] & \text{若自由度受約束}
\end{cases}
$$

---

### 5. 構件力恢復

對於每個構件 e：

**步驟 1：提取全域位移**

$$
\mathbf{d}_{\text{global}} = \begin{bmatrix}
D[3i] \\ D[3i+1] \\ D[3i+2] \\ D[3j] \\ D[3j+1] \\ D[3j+2]
\end{bmatrix}
$$

**步驟 2：轉換到局部座標**

$$
\mathbf{u}_{\text{local}} = \mathbf{T} \cdot \mathbf{d}_{\text{global}}
$$

**步驟 3：計算局部系統中的構件力**

$$
\mathbf{f}_{\text{local}} = \mathbf{k}_{\text{local}} \cdot \mathbf{u}_{\text{local}}
$$

**步驟 4：減去固端力**（若存在構件載重）

$$
\mathbf{f}_{\text{local}} \mathrel{-}= \mathbf{f}_{\text{FE,local}}
$$

**物理解釋**：

$$
\mathbf{f}_{\text{local}} = \begin{bmatrix}
N_1 \\ V_1 \\ M_1 \\ N_2 \\ V_2 \\ M_2
\end{bmatrix}
$$

其中：
- N₁, N₂：軸力（正值 = 拉力）
- V₁, V₂：剪力
- M₁, M₂：彎矩（逆時針為正）

**輸出的符號慣例**：
- **軸力**：N₂（節點 j 處的力，拉力為正）
- **起點剪力**：V₁
- **終點剪力**：V₂
- **起點彎矩**：M₁
- **終點彎矩**：M₂

**步驟 5：轉換回全域以計算反力**

$$
\mathbf{f}_{\text{global}} = \mathbf{T}^T \cdot \mathbf{f}_{\text{local}}
$$

$$
R_{\text{accum}}[3i] \mathrel{+}= \mathbf{f}_{\text{global}}[0]
$$

$$
R_{\text{accum}}[3i+1] \mathrel{+}= \mathbf{f}_{\text{global}}[1]
$$

$$
R_{\text{accum}}[3i+2] \mathrel{+}= \mathbf{f}_{\text{global}}[2]
$$

$$
R_{\text{accum}}[3j] \mathrel{+}= \mathbf{f}_{\text{global}}[3]
$$

$$
R_{\text{accum}}[3j+1] \mathrel{+}= \mathbf{f}_{\text{global}}[4]
$$

$$
R_{\text{accum}}[3j+2] \mathrel{+}= \mathbf{f}_{\text{global}}[5]
$$

---

### 6. 支承反力

**最終反力**（平衡）：

$$
\mathbf{R} = \mathbf{R}_{\text{accum}} - \mathbf{F}_{\text{applied}}
$$

對於每個支承節點 n：

$$
R_x = R_{\text{accum}}[3n] - F_{\text{applied}}[3n]
$$

$$
R_y = R_{\text{accum}}[3n+1] - F_{\text{applied}}[3n+1]
$$

$$
R_m = R_{\text{accum}}[3n+2] - F_{\text{applied}}[3n+2]
$$

其中 **F_applied** 僅包括直接節點載重，不包括構件衍生的固端力。

---

### 7. 任意點的內力

計算距節點 i 距離 x 處的 M(x), V(x), N(x)：

**初始化**（從節點力）：

$$
M(x) = -M_1 + V_1 \cdot x
$$

$$
V(x) = V_1
$$

$$
N(x) = N_1
$$

**加上 0 到 x 之間載重的貢獻**：

對於每個構件載重：

##### 距離 a 處的點載重

若 x > a：

$$
M(x) \mathrel{+}= P_y \cdot (x - a)
$$

$$
V(x) \mathrel{+}= P_y
$$

$$
N(x) \mathrel{+}= P_x
$$

##### 從 d₁ 到 d₂ 的分佈載重

有效段：[max(d₁, 0), min(d₂, x)]

若 end > start：

$$
\text{len} = \text{end} - \text{start}
$$

**插值強度**：

$$
w_{\text{start}} = w_1 + (w_2 - w_1) \frac{\text{start} - d_1}{d_2 - d_1}
$$

$$
w_{\text{end}} = w_1 + (w_2 - w_1) \frac{\text{end} - d_1}{d_2 - d_1}
$$

**合力**：

$$
W = \frac{w_{\text{start}} + w_{\text{end}}}{2} \cdot \text{len}
$$

**從 "start" 的形心距離**：

$$
\bar{x}_{\text{local}} = \frac{\text{len} \cdot (w_{\text{start}} + 2w_{\text{end}})}{3(w_{\text{start}} + w_{\text{end}})}
$$

**到 x 的力臂**：

$$
\text{lever} = (x - \text{start}) - \bar{x}_{\text{local}}
$$

**分解為分量**：

$$
W_x = W \cos(\theta_{\text{rel}})
$$

$$
W_y = W \sin(\theta_{\text{rel}})
$$

其中：

$$
\theta_{\text{rel}} = \theta_{\text{global}} - \theta_{\text{beam}}
$$

**更新**：

$$
M(x) \mathrel{+}= W_y \cdot \text{lever}
$$

$$
V(x) \mathrel{+}= W_y
$$

$$
N(x) \mathrel{+}= W_x
$$

---

### 8. 撓度（橫向位移）

**三次 Hermite 插值**（從節點位移）：

$$
v(x) = v_1 H_1(t) + \theta_1 L \cdot H_2(t) + v_2 H_3(t) + \theta_2 L \cdot H_4(t)
$$

其中：

$$
t = \frac{x}{L}
$$

**Hermite 形狀函數**：

$$
H_1(t) = 1 - 3t^2 + 2t^3
$$

$$
H_2(t) = t(1 - 2t + t^2) = t - 2t^2 + t^3
$$

$$
H_3(t) = 3t^2 - 2t^3
$$

$$
H_4(t) = t^2(t - 1) = t^3 - t^2
$$

**相對撓度**（減去剛體運動）：

$$
v_{\text{linear}}(t) = v_1(1 - t) + v_2 \cdot t
$$

$$
v_{\text{disp}}(x) = v(x) - v_{\text{linear}}(x)
$$

這將彎曲變形與剛性平移分離。

---

## 斷面性質計算

### 標準形狀

#### 1. 矩形

**輸入**：深度 d (mm)，寬度 b (mm)

**公式**：

$$
A = b \cdot d \quad (\text{m}^2)
$$

$$
I = \frac{b \cdot d^3}{12} \quad (\text{m}^4)
$$

#### 2. 空心矩形（方管）

**輸入**：外側 d, b (mm)，壁厚 t (mm)

**內部尺寸**：

$$
d_i = d - 2t, \quad b_i = b - 2t
$$

**公式**：

$$
A = bd - b_i d_i
$$

$$
I = \frac{bd^3 - b_i d_i^3}{12}
$$

#### 3. 圓形

**輸入**：直徑 d (mm)

**公式**：

$$
A = \frac{\pi d^2}{4}
$$

$$
I = \frac{\pi d^4}{64}
$$

#### 4. 空心圓（圓管）

**輸入**：外徑 d (mm)，壁厚 t (mm)

**內徑**：

$$
d_i = d - 2t
$$

**公式**：

$$
A = \frac{\pi (d^2 - d_i^2)}{4}
$$

$$
I = \frac{\pi (d^4 - d_i^4)}{64}
$$

#### 5. I 型（寬翼緣）

**輸入**：深度 d，翼板寬度 b，腹板厚度 t_w，翼板厚度 t_f (mm)

**內部尺寸**：

$$
h_{\text{inner}} = d - 2t_f
$$

$$
b_{\text{inner}} = b - t_w
$$

**公式**：

$$
A = 2b t_f + h_{\text{inner}} t_w
$$

$$
I = \frac{b d^3 - b_{\text{inner}} h_{\text{inner}}^3}{12}
$$

#### 6. C 型（槽鋼）

**與 I 型相同的公式**（對稱近似）

#### 7. 三角形

**輸入**：底邊 b，高度 d (mm)

**公式**：

$$
A = \frac{b \cdot d}{2}
$$

$$
I = \frac{b \cdot d^3}{36}
$$

---

## 視覺化與渲染

### 畫布座標系統

**螢幕到世界**：

$$
x_{\text{world}} = \frac{x_{\text{screen}} - \text{pan}_x}{\text{zoom}}
$$

$$
y_{\text{world}} = \frac{\text{pan}_y - y_{\text{screen}}}{\text{zoom}}
$$

**世界到螢幕**：

$$
x_{\text{screen}} = \text{pan}_x + x_{\text{world}} \cdot \text{zoom}
$$

$$
y_{\text{screen}} = \text{pan}_y - y_{\text{world}} \cdot \text{zoom}
$$

### 網格系統

**動態步長計算**：

高縮放（zoom ≥ 40 px/m）：

$$
\text{step} = \frac{0.5}{2^{\lfloor \log_2(\text{zoom}/40) \rfloor}}
$$

低縮放（zoom < 40 px/m）：

$$
\text{rawStep} = \frac{50}{\text{zoom}}
$$

$$
\text{magnitude} = 10^{\lfloor \log_{10}(\text{rawStep}) \rfloor}
$$

$$
\text{residual} = \frac{\text{rawStep}}{\text{magnitude}}
$$

$$
\text{step} = \begin{cases}
10 \cdot \text{magnitude} & \text{若 residual} > 5 \\
5 \cdot \text{magnitude} & \text{若 residual} > 2 \\
2 \cdot \text{magnitude} & \text{若 residual} > 1 \\
\text{magnitude} & \text{其他}
\end{cases}
$$

### 圖表縮放

**全域比例**（從所有構件計算）：

$$
\text{scale}_M = \frac{1.5}{\max |M(x)|}
$$

$$
\text{scale}_V = \frac{1.5}{\max |V(x)|}
$$

$$
\text{scale}_N = \frac{1.5}{\max |N(x)|}
$$

$$
\text{scale}_\delta = \frac{1.0}{\max |v(x)|}
$$

**顯示比例**（使用者可調整）：

$$
\text{offset}_{\text{pixels}} = \text{value} \times \text{scale} \times \text{diaScale} \times 40
$$

### 變形形狀渲染

**曲梁**（沿長度 40 步）：

對於每一步 i = 0 到 40：

$$
t = \frac{i}{40}
$$

$$
x = t \cdot L
$$

**撓度**：

$$
v_{\text{disp}}(x) = \text{(Hermite 插值)}
$$

**螢幕偏移**（垂直於梁）：

$$
\text{offset}_{\text{screen}} = -v_{\text{disp}}(x) \cdot \text{defScale} \cdot \text{zoom}
$$

**法向量**（垂直方向）：

對於水平梁（|dy| ≤ |dx|）：

$$
n_x = -\frac{dy}{L_{\text{screen}}}, \quad n_y = \frac{dx}{L_{\text{screen}}}
$$

對於垂直梁（|dy| > |dx|）：

$$
n_x = -\frac{dy}{L_{\text{screen}}}, \quad n_y = \frac{dx}{L_{\text{screen}}}
$$

**曲線點位置**：

$$
x_{\text{screen}} = x_{1,\text{screen}} + t \cdot \Delta x_{\text{screen}} + n_x \cdot \text{offset}_{\text{screen}}
$$

$$
y_{\text{screen}} = y_{1,\text{screen}} + t \cdot \Delta y_{\text{screen}} + n_y \cdot \text{offset}_{\text{screen}}
$$

---

## 單位慣例

| 參數 | 單位 | 說明 |
|-----------|------|-------------|
| **長度** | m | 節點座標、跨距長度 |
| **小長度** | cm | 斷面尺寸、厚度 |
| **力** | kN | 載重、反力、內力 |
| **彎矩** | kN·m | 彎矩 |
| **分佈載重** | kN/m | 線載重 |
| **應力** | kPa | 彈性模數 E |
| **面積** | m² | 斷面積 |
| **慣性矩** | m⁴ | 慣性矩 |
| **角度** | degrees | 載重角度、支承旋轉 |

---

## 瀏覽器相容性

* **Chrome/Edge**：v90+
* **Firefox**：v88+
* **Safari**：v14+
* **行動裝置**：iOS Safari、Chrome Mobile（啟用觸控控制）

**所需功能**：
- HTML5 Canvas
- ES6 JavaScript（箭頭函數、解構、模組）
- CSS Grid 和 Flexbox
- LocalStorage API
- FileReader API（用於專案載入）

---

## 已知限制

1. **僅限 2D 分析**：不考慮平面外效應
2. **線性彈性**：無材料非線性或塑性
3. **小變形**：不包括幾何非線性（P-Δ 效應）
4. **無自動網格**：使用者必須手動離散化曲線構件
5. **無時間歷程**：僅靜力分析
6. **有限載重類型**：無溫度載重、無移動載重
7. **無設計規範檢核**：純分析工具（無 ACI/Eurocode 合規性檢核）

---

## 參考文獻

### 教科書
* **Matrix Structural Analysis** by William McGuire, Richard H. Gallagher, Ronald D. Ziemian (2nd Edition)
* **Structural Analysis** by Aslam Kassimali (6th Edition)
* **Finite Element Procedures** by Klaus-Jürgen Bathe

### 規範
* **AISC Steel Construction Manual** (15th Edition) - 斷面性質
* **ACI 318-19** - 混凝土設計（僅參考材料性質）

### 演算法
* **Gaussian Elimination with Partial Pivoting** - Numerical Recipes (Press et al.)
* **Hermite Interpolation** - 標準有限元公式

---

## 支援

**開發者**：gacchuguts@gmail.com

---

**免責聲明**：本軟體作為教育和初步設計工具提供。使用者必須獨立驗證所有結果，並確保符合適用的規範和標準。作者不對錯誤、遺漏或使用後果承擔責任。最終設計批准請務必諮詢持照專業工程師。