# 股價預測與程式交易
### Team members : 陳律嘉、郭東輝、吳致傑
---
title: 202209_ML&FinTech_09
tags: 202209_ML&FinTech
---


# 量化交易與機器學習濾網建置

## 1. Motivation

![](https://i.imgur.com/XWIdArB.jpg)

隨著減碳議題逐漸被重視，台灣也預計在2040年禁售燃油機車，與國際攜手往2050零碳排的目標邁進，可知電動車產業成為未來發展的重點，另外**電動車產業彼此之間可以聯想到有高度的相關性(題材)，因此我們想透過機器學習的技術對目標公司的股票進行股價預測**，希望進一步成為投資人的參考依據或指標。

本報告的核心價值在於**利用機器學習中不同的演算法**（ex: random forest、xgboost、隨機森林、SVM）來比較及預測未來的股票走勢，再以模型訓練的結果來**修正原始交易策略**，使得Return、Mdd & CalmarRatio獲得明顯改善，以此建構出一套戰勝市場的交易策略。


## 2. Data 
**預測標的: 2308 台達電**($y$)
我們想去<font color="blue">預測隔日股價上漲的機率</font>($y_{t+1}$)，以下Table為模型$x_t$

|Notation|Description|Type|
|--|--|---|
| $x_{t,1}$|2360 致茂調整股價 at day $t$|Price & Return|
| $x_{t,2}$|3665 貿聯KY調整股價 at day $t$|Price & Return|
| $x_{t,3}$|1536  和大調整股價 at day $t$|Price & Return|
| $x_{t,4}$|1503士電調整股價 at day $t$|Price & Return|
| $x_{t,5}$|4721 美琪瑪調整股價 at day $t$|Price & Return|
| $x_{t,6}$|2371 大同調整股價 at day $t$|Price & Return|
| $x_{t,7}$|1537 廣隆調整股價 at day $t$|Price & Return|
| $x_{t,8}$|1723 中碳調整股價 at day $t$|Price & Return|
| $x_{t,9}$|6509 聚和調整股價 at day $t$|Price & Return|
| $x_{t,10}$|8183 精星調整股價 at day $t$|Price & Return|
| $x_{t,11}$|1533 車王電調整股價 at day $t$|Price & Return|
| $x_{t,12}$|TWII 台股大盤加權指數 at day $t$|Price & Return|
| $x_{t,13}$|S&P500 標準普爾指數 at day $t$|Price & Return|
| $x_{t,14}$|法人買賣超幅度(外資) at day $t$|Volume|
| $x_{t,15}$|法人買賣超幅度(投信) at day $t$|Volume|
| $x_{t,16}$|法人買賣超幅度(自營) at day $t$|Volume|
| $x_{t,17}$|台達電成交量 at day $t$|Volume|
| $x_{t,18}$|MA20 at day $t$|Technical|
| $x_{t,19}$|MA60 at day $t$|Technical|
| $x_{t,20}$|KD(K) at day $t$|Technical|
| $x_{t,21}$|KD(D) at day $t$|Technical|
| $x_{t,22}$|MACD at day $t$|Technical|
| $x_{t,23}$|RSI at day $t$|Technical|

### 資料起訖時間:
| **類別** | **股價** | **報酬率** |
| -------- | -------- | -------- |
| 機器學習訓練+測試期     | 2011/7/18~2020/9/30     | 2011/7/19~2020/9/30     |
|     交易策略期間    |       2020/10/5~2022/10/7   |        2020/10/5~2022/10/7|


附註:
- MA為移動平均線，代表過去一段時間的平均成交價格
$$MA_{20}=\tfrac{S_{1}+S_{2}+...S_{20}}{20}$$
$$MA_{60}=\tfrac{S_{1}+S_{2}+...S_{60}}{60}$$

- MACD指標:
(1)EMA: 依據不同天，時間較近的資料給予較高的權重，所計算出的移動平均線
(2)離差值DIF是利用短期與長期的EMA相減計算出來的
(3)計算出DIF後，再取DIF的移動平均，就是 MACD 線
$$DIF=EMA_{12}-EMA_{26}$$
$$MACD=EMA(DIF,9)$$

- RSI指標:
測量現在股價是強還是弱，數字越大表示股價愈強，本報告以14日做計算

$$RSI=\tfrac{(一段時間內)股票漲幅平均值}{(一段時間內)股票漲幅平均值+(一段時間內)股票跌幅平均值}\times100$$ 



- KD指標:
(1)K值為「快速平均值」，對股價變化的反應較快速
(2)D值為「慢速平均值」，對股價變化的反應較不靈敏

$$K_t=\tfrac{2}{3}\times K_{t-1}+\tfrac{1}{3}\times RSV_t$$

$$D_t=\tfrac{2}{3}\times D_{t-1}+\tfrac{1}{3}\times K_t$$


### :apple:EDA(探索式資料分析)
在資料科學上，首先應執行資料探勘，運用視覺化或基本統計等工具，來對資料有個初步的認識，以利後續對資料進行複雜、嚴謹的分析，亦即所謂的三大步驟，**<font color="#f00">檢查資料、觀察特徵到構想模型</font>**，良好並有效率的資料探勘分析幫助我們釐清問題，並執行有意義的研究。

#### 一、熱力圖
#### 股價:
![](https://i.imgur.com/kvA8cbu.png)


從股價熱力圖可觀察到幾個現象:
 - **各大類股皆有相當程度的關聯性**: 探究原因乃為同類型、同樣相關行業的股票通常都會有齊漲齊跌的現象，乃為股票同質性。
 - **移動平均與台達電或是各大類股也有一定程度的相關**: 移動平均是由台達電股價資料計算而成，而台達電股價又與類股走勢相關，因此移動平均也會與類股走勢相關。
 - **技術指標與投信、外資的法人買賣超呈現普通程度的相關**: 可能是買賣超幅度有某一程度的力量來影響股票走勢，進一步帶動其相關性，不過技術指標間還是有不同程度的相關性。
 - **台股大盤指數與標準普爾指數皆有顯著相關性**: 推測大盤的變動有可能為我們準確預測的精隨之一，而與本報告欲預測的台達電也有頗高的線性相關。


#### 報酬率:
![](https://i.imgur.com/f2Q4pGK.png)

 - **類股的股價資料在轉換為報酬率後，發現仍有輕度的線性相關**。

#### 二、漲跌次數

![](https://i.imgur.com/y85Py5n.png)

**台達電漲跌次數分配<font color="#f00">並無不平衡現象</font>，模型不會學習到大量漲或跌的資料，而產生過度擬合現象**

#### 三、結論
從共變異數熱力圖可以發現，在**相關電動車概念股中，彼此之間皆高度相關**，故在線性相關上，相關概念股在某種程度上可以解釋台達電股價，但股價之間相關並不代表在隨機性更高的漲跌之間會具有顯著解釋力，故未來主要著重在非線性關係上準確率之良窳

## 3. Formulation

$S_t$: 股票在時間點$t$的價格
$y_t=I_{\{S_t>S_{t-1}\}}$: 台達電之隔日股價是否上漲(以0、1表示)

$x=(x_1,\ldots,x_{23})$, $p=23$. (此處已在Data的部分提及，故不再加以贅述)

我們透過所學**金融知識、EDA**(探索式資料分析)篩選出合適的 $x$ 並預測 $y$，我們使用機器學習中
**分類樹、隨機森林、XGBoost、SVM**等方法來進行分類問題，期望能藉由預測漲跌趨勢，進行
後續交易策略的優化，帶來更加低風險、高報酬的績效。

## 4. Analysis
### 資料型態為price

| Method | Confusion matrix | 
| ------ | ---------------- | 
|分類樹|$\begin{bmatrix}81 & 146 \\67 & 160\end{bmatrix}$| 
隨機森林 |$\begin{bmatrix}174 & 53 \\164 & 63\end{bmatrix}$|
XGboost|$\begin{bmatrix}132 & 95 \\119 & 108\end{bmatrix}$|
SVM|$\begin{bmatrix}92 & 180 \\73 & 109\end{bmatrix}$|




## 模型比較


| 績效 | 分類樹 |  隨機森林   |  XGboost   | SVM   |
| ------ |:------ | --- | --- | --- |
| Accuracy  |**0.53084**                              | 0.52203 |  0.52863 |    0.44273 |     |
| F1 Score  | **0.60038** | 0.36735                               | 0.50233 |     0.46285 |     |
| Recall    | **0.70485** | 0.27753                               |0.47577 |     0.59890 |     |
| Precision | 0.52288                               | **0.54310** |  0.53202 |    0.37716|
### 資料型態為return



| Method | Confusion matrix | 
| ------ | ---------------- | 
|分類樹|$\begin{bmatrix}127 & 99 \\98 & 129\end{bmatrix}$| 
隨機森林 |$\begin{bmatrix}194 & 32 \\178 & 49\end{bmatrix}$|
XGboost|$\begin{bmatrix}161 & 65 \\130 & 97\end{bmatrix}$|
SVM|$\begin{bmatrix}96 & 147 \\80 & 130\end{bmatrix}$|

## 模型比較
| 績效 | 分類樹 |  隨機森林   |  XGboost   |SVM   |
| ------ |:------ | --- | --- | --- |
|  Accuracy  | 0.56512    |  0.53642    |   **0.56954** |0.49890|
| F1 Score   |  **0.56703**   | 0.31818    | 0.49871    |0.53388|
| Recall   |   0.56828   |0.21586   |   0.42731 |**0.61905**|
| Precision   | 0.56579     | **0.60494**   |  0.59877  |0.46931|



## Conclusion
### 資料形式為price
- <font color="#f00">**分類樹**</font>在預測股票漲跌中，F1 Score為0.60038，為所有模型中表現最佳，且在<font color="#f00">**預測為漲的準確度**</font>高達0.70485，為為資料形式price模型中看漲最佳表現
- <font color="#f00">**分類樹**</font>平均準確率為0.53084，為資料形式price模型中最佳表現
- <font color="#f00">**隨機森林**</font>在<font color="#f00">**看跌預測中準確度**</font>為0.76，為所有資料形式price模型中看跌最佳表現，但漲預測準確度低，故總準確度僅為0.52203
- <font color="#f00">**XGboost**</font>平均準確度為0.52863，僅次於分類樹模型(資料形式price)

### 資料形式為return

- <font color="#f00">**分類樹**</font>在預測股票漲跌中，F1 Score為0.56703，為所有資料形式return模型中表現最佳，且看漲與看跌機率差異不大(看跌:0.5545，看漲:0.56828)，平均優於其他模型
- <font color="#f00">**SVM**</font>在<font color="#f00">**看漲預測準確度**</font>為0.619為所有資料形式return看漲中有最好表現
- <font color="#f00">**隨機森林**</font>在<font color="#f00">**看跌預測準中準確度**</font>為0.85，為所有資料形式return看跌預測中表現最佳，但漲預測準確度低，故總準確度僅為0.53642
- <font color="#f00">**XGboost**</font>平均準確度為0.56954，為所有資料形式return模型中最佳表現
### 總結
- <font color="#f00">**資料形式為return分類樹**</font>在預測股票漲跌中有最佳預測表現
- <font color="#f00">**資料形式為return隨機森林**</font>在看跌預測有最佳表現
- <font color="#f00">**資料形式為price分類樹**</font>在看漲預測有最佳表現
- 不管資料形式price或return，<font color="#f00">**分類樹與XGboost**</font>皆有較佳平均準確度

#### 附註
- 由於分類樹本身的預測是藉由「一連串的判斷，把input分配到樹中的某一個leaf node，並依據同個leaf node裡面比例最高類別當成預測」，故並沒有機率性質的預測，**故在此機率預測方法為：用以最終在的leaf node之中漲跌比率來當成預測的機率。**
- 由於SVM是藉由每個點對Decision Boundary的位置，故以離Decision Boundary越近，我們模型的信心越低，離Decision Boundary越遠，我們模型的信心越高為概念，**在SVM結果加入Sigmoid**，將原本不具備機率性質的預測轉換為符合機率的公理

|           | 實際Yes          | 實際No |
| ----------------- |:------------- | ---------- |
|     預測Yes     | TP(True Positive)|     FP(False Positive)      |
| 預測No  | FN(False Negative)   |       TN(True Negative)     |


**準確度(Accuracy) = (TP+TN)/(TP+FP+FN+TN)**
**精確度(Precision) = TP/(TP+FP)**
**召回率(Recall) = TP/(TP+FN)**
**F1-score = 2 * Precision * Recall / (Precision + Recall)**

---
## 交易策略
### :pencil: 主邏輯(原策略)

```
買進: RSI_12 > 50 + RSI_6 > RSI_12(黃金交叉) + close > MA 
放空: RSI_12 < 50 + RSI_6 < RSI_12(死亡交叉) + close < MA 

賣出: RSI_12 < 50 + close < MA
回補: RSI_12 > 50 + close > MA

停損停利: 5%

回測區間: 2020/10/05~2022/10/07
資金管理: 設一開始持有本金$1000元，每一次交易都固定投入$100
手續費: 0.1425% (進出各課一次)
證交稅: 0.3% (平倉時課一次)

RSI_12: 12天計算之RSI
RSI_6: 6天計算之RSI
MA: 110天日內移動平均線
附註: 在部位全部平倉前，不會開啟新的倉位(無加減碼)
```
### 回測結果

**紅線**:未計算交易成本之損益曲線

**藍線**:有計算交易成本之損益曲線(<font color="#f00">手續費+稅</font>)

**無考慮交易成本下，獲利有不斷創高的情況，代表主策略的邏輯沒有太大問題，但是在考慮交易成本下，獲利被吃掉許多，可知這個策略是有很多優化空間的。**

![](https://i.imgur.com/dFno445.png)



### Profit & Drawdown
**可發現在盤整區域與空頭區段出現dd擴大的情況(紅色區域)**
![](https://i.imgur.com/xHSX192.png)



### 交易落點分析
觀察交易策略的問題得以進一步優化策略
- 多單的勝率 = 63.16%
- 空單的勝率 = 36.36%
* **可知做空的點位差強人意，有許多改進的空間**
![](https://i.imgur.com/81XsMZJ.png)


### 績效評估
**可以將重點擺在 CalmarRatio = 年化報酬率 / 最大回落比例**
 (反映投資人面臨資產價格大幅波動時的心理痛苦程度)

| 績效 | 意義 | 基本策略 |
| ------ |:--------|:------ |
| Profit |  累積獲利 | $40.8   | 
| **Return** |   累積報酬 | 4.08% |
|**Mdd**    |     權益最大回落程度(%) | 2.39% |
| **Calmar Ratio** |  Return / Mdd | 1.71 |
| Trade Times |    總交易次數 | 41  |
| Win Rate    |  交易獲利次數 / 總交易次數 | 46.78%  |
| Profit Factor |   總獲利 / 總損失 | 1.43  |
| Win Loss Ratio |  平均獲利 / 平均損失 | 1.5  |


### 參數最佳化  →  K = 0.05, length(MA) = 110
![](https://i.imgur.com/YCB8Kgv.png)

---

## 加入濾網策略與回測整體績效

### **如何修正**
我們希望利用機器學習模型來<font color="#f00">**預測明日股價漲跌的機率**</font>，**藉由模型結果來輔助我們判斷做空進場的時機點，過濾掉因市場的雜訊所釋出的假訊號**，進而找出更加的空單進場點位，不只提高作空的勝率，也提升了整體績效。

**0.7為主觀認定**: 我們認為當模型預測明日下跌機率超過0.7時，可以更加確定空方力道，因此在滿足主策略與此條件時才會進行空單佈局。


買進: RSI_12 > 50 + RSI_6 > RSI_12(黃金交叉) + close > MA 
放空: RSI_12 < 50 + RSI_6 < RSI_12(死亡交叉) + close < MA + <font color="#f00">**隔日下跌機率 > 0.7 </font>**
(其於假設皆與之前一致，可至原策略回顧)


---
### 整體績效之比較
由於**XGboost**與**分類樹**有較佳預測的平均準確度，因此主要使用這兩種模型進行優化

| 績效 | <font color="#f00">基本策略</font> |XGboost(price)|分類樹(price)|XGboost(return)|:trophy:分類樹(return)|
| ------ |------|-----|------|------|------|
| Profit |  $40.8 |$52.63|$30.4|$52.53|$58.73|
| Return |  4.08%|5.26%|3.04%|5.25%|5.87%|
|**Mdd**    |  <font color="#f00">**2.39%**</font>|1.46% |2.51%|2.31%|**1.46%**|
| **Calmar Ratio** | <font color="#f00">**1.71**</font>|3.61 |1.21|2.27|**4.03**|
| Trade Times |  41|18| 32| 38 |29|
| **Win Rate**    |  <font color="#f00">**46.78%**</font>|66.67% |53.12% | 52.63%|**62.07%**|
| Profit Factor | 1.43|2.51|1.4 |1.66 |2.03|
| Win Loss Ratio | 1.5|1.26| 1.23|1.5 |1.24|

**與前述觀察結果一致，結果為分類樹(return)優化結果表現最為亮眼，成功控制住Mdd，勝率從46%提高至62%，Calmar Ratio也提升至4倍以上，<font color="#f00">確實排除了不佳的做空點位</font>(雜訊)。**

---
### 績效差異(price)


 - 無交易成本獲利(red) : $63.46
 - 有交易成本獲利(blue) : $52.63

![](https://i.imgur.com/E89ayCA.png)

![](https://i.imgur.com/ht1l354.png)

多單勝率 : 66.67%
**無放空交易**(皆不滿足明日下跌機率0.7的條件)

![](https://i.imgur.com/cNOZvSv.png)

在此模型中，確實有讓獲利持續創高，但因為XGboost(price)預測明日下跌機率皆沒有大於70%，搭配濾網的策略放棄了所有放空機會，只佈入多單，此結果已經違背濾網設置的初衷，且有交易次數過少的問題。

### 分類樹

 - 無交易成本獲利(red) : $49.34
 - 有交易成本獲利(blue) : $30.40

![](https://i.imgur.com/Z6yg6n6.png)

![](https://i.imgur.com/jTB9A68.png)

 - 多單勝率 : 64.71%
 - 空單勝率 : 40%

![](https://i.imgur.com/xwSraUW.png)

在此模型中，獲利沒有持續創高，且可看出若考慮交易成本，獲利甚至出現減少的趨勢，雖然空單勝率提高了，但卻影響到多單的勝率，這是因為策略採取單一部位進場，在加入濾網條件後，還影響到了多單進場的時機。

---
### 績效差異(return)
### XGboost

 - 無交易成本獲利(red) : $75.09
 - 有交易成本獲利(blue) : $52.53

![](https://i.imgur.com/4iAZDP6.png)

![](https://i.imgur.com/vTSkrMZ.png)

 - 多單勝率 : 66.67%
 - 空單勝率 : 40%

![](https://i.imgur.com/m0wZOXl.png)

在此模型中，獲利有持續性創高，但是考慮交易成本的損益與無考慮交易成本的損益差距越來越大，可知交易成本吃掉了很多獲利，仍有很多的優化空間。


### 分類樹

 - 無交易成本獲利(red) : $76.03
 - 有交易成本獲利(blue) : $58.73

![](https://i.imgur.com/zh5XvI6.png)

![](https://i.imgur.com/mFuOdmk.png)

 - 多單勝率 : 66.67%
 - 空單勝率 : 54.55%

![](https://i.imgur.com/ThNKZmv.png)

在此模型中，不只看出獲利有明顯創高，DD也有控制住，而交易成本也沒有吃掉太多利潤，可知用分類樹(return)模型來進行交易策略優化，有較好的效果。



##  Conclusion

**以回測績效來說，分類樹(return)模型能夠最有效率的優化原本的交易策略，達到Calmar Ratio 超過4倍的佳績，有最高的風險報酬比。雖然沒有進一步進行樣本外回測(檢驗策略的實際可行度)，但我們成功挖掘出機器學習用於優化交易策略的可能性，我們認為這才是此專案最大的貢獻**。
