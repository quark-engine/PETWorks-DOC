+++++++++++++++++++++++++++++++++++++++
資料獲利力評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估資料是否滿足 獲利力（profitability）。更多獲利力之說明，詳見 `此處 <#id4>`_ 。

我們以 ``data/delta.csv`` 作爲原始資料表， ``data/delta_anonymized.csv`` 作為範例資料表， ``data/delta_hierarchy`` 作為資料階層定義，以及 ``attributeTypes`` 作為屬性型態定義，展示如何透過 PETWorks-framework 框架判斷此指標。

在以下程式碼中，我們透過 API ``PETValidation(origin, anonymized, "profitability", dataHierarchy, attributeTypes, allowAttack, adversaryCost, adversaryGain, publisherLost, publisherBenefit)`` ，以上述資料、“profitability” 字串以及 allowAttack 、adversaryCost 、adversaryGain 、publisherLost 與 publisherBenefit 作爲參數，判斷資料是否滿足獲利力。

再來，我們透過 API ``report(result, format)`` ，以上述評估結果與 “json” 字串作爲參數，將評估結果以 JSON 格式印出。

範例程式碼: profitability.py
-----------------------------

.. code-block:: python

    from PETWorks import PETValidation, report
    from PETWorks.attributetypes import QUASI_IDENTIFIER, INSENSITIVE_ATTRIBUTE

    origin = "data/delta.csv"
    anonymized = "data/delta_anonymized.csv"
    dataHierarchy = "data/delta_hierarchy"

    attributeTypes = {
        "zip": QUASI_IDENTIFIER,
        "age": QUASI_IDENTIFIER,
        "nationality": QUASI_IDENTIFIER,
        "salary-class": INSENSITIVE_ATTRIBUTE
    }

    result = PETValidation(
        origin,
        anonymized,
        "profitability",
        dataHierarchy=dataHierarchy,
        attributeTypes=attributeTypes,
        allowAttack=True,
        adversaryCost=4,
        adversaryGain=300,
        publisherLost=300,
        publisherBenefit=1200
    )
    report(result, "json")


輸出結果
--------

.. code-block:: text
    
    $ python3 profitability.py
    {
        "allow attack": true,
        "adversary's cost": 4,
        "adversary's gain": 300,
        "publisher's loss": 300,
        "publisher's benefit": 1200,
        "profitability": true
    }


評估指標之定義
--------------

獲利力是一種隱私保護力指標，表示資料表發布後，發布者獲得之收益需大於發布者承受之風險。該要件以資料列為單位獨立評估。若資料列的發布價值越高，發布者收益越大；若資料列被成功再識別的機率越高，或隱私資料外洩衍生的損失越高，發布者風險越大。

指標另有 **不容許再識別攻擊發生** 之嚴格變種。該變種額外要求，攻擊者進行再識別攻擊之收益期望值，不得大於進行再識別攻擊之成本，即攻擊者無利可圖，從而達成更嚴格的隱私保護強度。

評估指標之判斷
---------------

評估獲利力指標，需要給定以下四種背景參數：

+ 攻擊者成本（adversary's cost）：攻擊者對一個資料列發動再識別攻擊付出之成本。
+ 攻擊者收益（adversary's gain）：攻擊者對一個資料列成功再識別攻擊獲得之利益。
+ 發布者損失（publisher's lost）：一個資料列被成功再識別攻擊，發布者受到之損失。
+ 發布者收益（publisher's benefit）：一個資料列被發布，發布者獲得之利益。


參考 [1]_ 與 [2]_ ，判斷獲利力指標之基礎定義有三個步驟：

1. 計算各資料列帶給攻擊者之收益期望值。
2. 參考步驟 1 之收益期望值，計算各資料列帶給發布者之風險。
3. 參考步驟 2 之風險，判斷發布者獲得之收益是否大於承受之風險。

而欲判斷獲利力指標之嚴格變種，還需一個步驟：

4. 參考步驟 1 之收益期望值，判斷攻擊者進行再識別攻擊之成本是否大於獲得之收益期望值。


接下來將以下列「範例資料表」說明判斷過程。

**範例資料表**

+---------------------+-------------------------+
| .. centered::  編號 | .. centered::  出生年   | 
+=====================+=========================+
| .. centered::  1    | .. centered::  197*     | 
+---------------------+-------------------------+
| .. centered::  2    | .. centered::  198*     |  
+---------------------+-------------------------+
| .. centered::  3    | .. centered::  198*     |  
+---------------------+-------------------------+

假設「範例資料表」之「出生年」為個體背景資訊，若欲判斷「範例資料表」是否滿足以下背景參數設定，

+ 攻擊者成本 為 4 元
+ 攻擊者收益 為 300 元
+ 發布者損失 為 300 元
+ 發布者收益 為 1200 元

之 基礎定義 或 嚴格變種 獲利力指標，可以如下進行。

**STEP 1：計算各資料列帶給攻擊者之收益期望值。**

資料列帶給攻擊者之收益期望值公式如下：

.. math :: 
      \begin{equation}
      \begin{aligned}
    資料列帶給攻擊者之收益期望值 &= 資料列被成功再識別攻擊機率 \times 攻擊者收益\\
    &= \frac{1}{資料表中具有相同個體背景資訊之資料列數} \times 攻擊者收益\\
    \end{aligned}
    \end{equation}


以「範例資料表」 **編號 1 資料列** 為例，在「範例資料表」中具有相同個體背景資訊的資料列共有 **一個** ，且攻擊者收益為 300 元，則資料列帶給攻擊者之收益期望值為 300 元。


.. math :: 
      \begin{equation}
      \begin{aligned}
    編號 1 資料列帶給攻擊者之收益期望值 &= 編號 1 資料列被成功再識別攻擊機率 \times 攻擊者收益\\
    &= \frac{1}{「範例資料表」中具有編號 1 資料列相同個體背景資訊之資料列數} \times 攻擊者收益\\
    &= \frac{1}{1} \times 300 = 300
    \end{aligned}
    \end{equation}

同理，得到 **編號 2、3 資料列** 帶給攻擊者之收益期望值為 150 元。

**STEP 2：參考步驟 1 之收益期望值，計算各資料列帶給發布者之風險。**

若資料列帶給攻擊者之收益期望值小於成本，攻擊者無利可圖，則資料列帶給發布者之風險為 0 元。反之，則資料列帶給發布者之風險公式如下：

.. math :: 
      \begin{equation}
      \begin{aligned}
    資料列帶給發布者之風險 &= 資料列被成功再識別攻擊機率 \times 發布者損失\\
    &= \frac{1}{資料表中具有相同個體背景資訊之資料列數} \times 發布者損失\\
    \end{aligned}
    \end{equation}


以「範例資料表」 **編號 1 資料列** 為例，其帶給攻擊者之收益期望值 300 元大於攻擊者成本 4 元，攻擊者有利可圖，故資料列帶給發布者之風險非 0 元。而在「範例資料表」中具有相同個體背景資訊的資料列共有 **一個** ，且發布者損失為 300 元，則資料列帶給發布者之風險為 300 元。


.. math :: 
      \begin{equation}
      \begin{aligned}
    編號1 資料列帶給發布者之風險 &= 編號1 資料列被成功再識別攻擊機率 \times 發布者損失\\
    &= \frac{1}{「範例資料表」中具有編號 1 資料列相同個體背景資訊之資料列數} \times 發布者損失\\
    &= \frac{1}{1} \times 300 = 300
    \end{aligned}
    \end{equation}


同理，得到 **編號 2、3 資料列** 帶給發布者之風險為 150 元。




**STEP 3：參考步驟 2 之風險，判斷發布者獲得之收益是否大於承受之風險。**

再來，確認發布者獲得之收益，是否大於步驟 2 計算之風險。在此例中發佈 **編號 1 資料列** 帶給發佈者收益為 1200 元，大於帶給發佈者之風險 300 元；發佈 **編號 2、3 資料列** 各帶給發佈者收益為 1200 元，大於帶給發佈者之風險 150 元。因此，「範例資料表」滿足獲利力指標之基礎定義。




**STEP 4：參考步驟 1 之收益期望值，判斷攻擊者進行再識別攻擊之成本是否大於獲得之收益期望值。**

最後，確認攻擊者進行再識別攻擊之成本，是否大於步驟 1 計算之收益期望值。在此例中發佈 **編號 1 資料列** 帶給攻擊者成本為 4 元，不大於帶給攻擊者之收益期望值 300 元；發佈 **編號 2、3 資料列** 各帶給攻擊者成本為 4 元，不大於帶給攻擊者之收益期望值 150 元。因此，攻擊者有利可圖，「範例資料表」不滿足獲利力指標之嚴格變種。


參考資料
---------

.. [1] Z. Wan et al., “A Game Theoretic Framework for Analyzing Re-Identification Risk,” PLOS ONE, vol. 10, no. 3, p. e0120592, Mar. 2015, doi: 10.1371/journal.pone.0120592.

.. [2] F. Prasser et al., “An Open Source Tool for Game Theoretic Health Data De-Identification.,” American Medical Informatics Association Annual Symposium, vol. 2017, pp. 1430–1439, Jan. 2017.
