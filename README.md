# DOTA Game Result Prediction in Machine Learning

## I. Introduction
- 作為近十年來蠻熱門的遊戲, 每年的世界大賽都吸引了世界各地人的關注, 而關於lol比賽的各種數據庫也逐漸完善, 最近甚至出現了對2020年世界大賽有100%準確率的[SenpaiAI](http://www.gzmjhzs.com/lol/1620152106.html), 但是觀察他的原文後, 發現他訓練過程其實準確率只有64%, 於是我就想有沒有機會利用這學期學的各種技巧來模擬這個結果, 預測BO1的勝負

## II. Data Collection
- 關於數據來源, 我選擇用爬蟲的方式來取得, 然後觀察了各種比賽的數據網後, 選擇了[gol.gg](https://gol.gg/esports/home/)當作我的數據來源, 蒐集了自 2014 - 2021 的所有比賽數據, 爬蟲程式跑了兩天左右才跑完
    
![](https://i.imgur.com/Zpm78Cr.png)
- 一場比賽的數據大約是長這樣, 數據種類十分的多, 所以我先人工篩選了一些我覺得對比賽比較有影響的數據下來, 總共有以下幾項
    1. Tower * 2
    2. Dragon * 2
    3. Baron * 2
    4. kills * 10
    5. deaths * 10
    6. assists * 10
    7. cs / min. * 10
    8. gold / min (gpm) * 10
    9. gold% * 10
    10. vision score / min (vspm) * 10
    11. wards placed / min (wpm) * 10
    12. wards destroyed / min  (wcpm)* 10
    13. Detector Wards Placed(紅眼數) / min (wcpm) * 10
    14. vs% * 10
    15. DPM * 10
    16. DMG% * 10
    17. GD@15 * 10
    18. CSD@15 * 10
    19. XPD@15 * 10
    20. DT * 10
    21. Game time * 1
    22. Game result * 1
- 而就像上圖所示, 不是所有的feature都會有值, 可能會有一些missing value, 所以我在爬蟲的時候也會把有missing value的比賽略過, 最後總共收集了 **20846** 場比賽數據與 **9300** 場略過的比賽

## III. Data Visualization
- 收集到的數據, 總共 **20846** 場比賽

![](https://i.imgur.com/99YA1AQ.png)

- 比賽勝負分布, 可以看到藍方獲勝稍微多一點, 但是也還在合理範圍

![](https://i.imgur.com/rEJ2bOV.png)

- 比賽日期分布, 可以看到雖然我爬到2014年, 但是2017年以前的數據一個都沒有, 代表早期的數據可能不太完整, 而每年的數據量也隨著最近幾年越來越多

![](https://i.imgur.com/dYldQUd.png)

- 比賽時長分布

![](https://i.imgur.com/C6uaTf9.png)

## IV. Preprocessing
- 因為我做的事賽前預測, 然後我得到的實際上是賽後數據, 所以我還要去做一些轉換才能得到training data, 對於每一場比賽假設 team0 v.s. team1
    - feature: 
        1. team0在藍方過去3場的平均數據
        2. team0在紅方過去3場的平均數據
        3. team1在藍方過去3場的平均數據
        4. team1在紅方過去3場的平均數據
        5. 將1-4攤平成一維, 總共feature數為386
    - label:
        - 比賽勝負 (0 or 1)
    - 哪一隊是team0哪一隊是team1是隨機sample的
- 因為是取兩隊各在紅藍方的過去三場的數據, 所以每個賽季初還沒打滿3場時的數據我會不採計, 這樣的話數據量從原本的 20000 多筆又降到 **14700** 筆
- 再考慮到我的input有點對稱性, 我可以將team0跟team1的feature顛倒放來得到新的一組feature, 變成 team1 v.s. team0, 而我測試這跟 team0 v.s. team1 的結果有時候可能會不一樣, 所以我接著測試validation的方法都會用兩種排列組合的結果機率相加來取得
    - **result = argmax(prob(team0, team1) + prob(team1, team0))**

## V. Models
- 我用了三種不同的model去預測上面preprocessing完的data
    1. Naive Bayes Classifier
    2. K Nearest Neighbor classifier
        - choose k = 1000
    4. Deep Neuron Network
        ![](https://i.imgur.com/1gfACsj.png)
- training : validation = 4 : 1

## VI. Results
- Naive Bayes Classifier

![](https://i.imgur.com/ck9AUWD.png)

- K Nearest Neighbor

![](https://i.imgur.com/sCHMj34.png)

- Deep Neuron Network

![](https://i.imgur.com/RuI9Ooj.png)

- 此外我還另外做了一個實驗, 假設model預測出來的輸贏機率如果是(0.51, 0.49), 代表model其實也不太確定到底哪一隊會贏, 我們說不定可以人為的去把這些他也不太確定的比賽取出來變成undecidable, 所以我有設定一個threshold讓每次結果中比較大的機率如果小於這個值就變成undecidable, 下面是threshold的值對應到undecidable rate以及相對應剩下的資料我們拿去算accuracy的值

![](https://i.imgur.com/7FofkrI.png)

- 可以看到如果設threshold = 0.65, 我們可以捨棄掉50%不太確定的資料, 那剩下的50%資料會擁有進70%的準確率, ~~我覺得這種性質很適合拿去買運彩, 因為我們可以自己決定要買哪一場比賽~~, 當然如果繼續改良模型與方法或者是新增更多資料, 未來一定也會有更好的結果

## VII. Conclusion
- 從上面的Results可以看到, 準確率最高的是DNN模型的63%, 非常接近我參考的原文ai的64%, 可以相信他應該也是用類似的數據庫跟方法做出來的, 當然我的預測只有做BO1(一戰定勝負)的, 同樣的實驗也可以做在BO3(三戰兩勝)與BO5(五戰三勝), 相信出來的結果會比BO1更準確
