+++++++++++++++++++++++++++++++++++++++
資料 δ-存在性評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估資料是否滿足 :math:`\delta`-存在性（:math:`\delta`-presence）。更多 :math:`\delta`-存在性之說明，詳見 `此處 <#id3>`_ 。

我們以 ``data/delta.csv`` 作爲原始資料表， ``data/delta_anonymized.csv`` 作為去識別化資料表， ``data/delta_hierarchy`` 作為資料階層定義，以及 ``attributeTypes`` 作為屬性型態定義，展示如何透過 PETWorks-framework 框架判斷此指標。

在以下程式碼中，我們透過 API ``PETValidation(origin, anonymized, "d-presence", dataHierarchy, attributeTypes, dMin, dMax)``，以上述資料、“d-presence” 字串，以及 dMin 與 dMax 作爲參數，判斷資料是否滿足 :math:`\delta`-存在性。

再來，我們透過 API ``report(result, format)``，以上述評估結果與 “json” 字串作爲參數，將評估結果以 JSON 格式印出。

範例程式碼: d-presence.py
-------------------------

.. code-block:: python

    from PETWorks import PETValidation, report
    from PETWorks.attributetypes import SENSITIVE_ATTRIBUTE, QUASI_IDENTIFIER


    origin = "data/delta.csv"
    anonymized = "data/delta_anonymized.csv"
    dataHierarchy = "data/delta_hierarchy"

    attributeTypes = {
        "zip": QUASI_IDENTIFIER,
        "age": QUASI_IDENTIFIER,
        "nationality": QUASI_IDENTIFIER,
        "salary-class": SENSITIVE_ATTRIBUTE
    }

    result = PETValidation(
            origin, anonymized, "d-presence", dataHierarchy=dataHierarchy, attributeTypes=attributeTypes, dMin=1/2, dMax=2/3
        )
    report(result, "json")


輸出結果
--------

.. code-block:: text
    
    $ python3 d-presence.py
    {
        "dMin": 0.5,
        "dMax": 0.6666666666666666,
        "d-presence": true
    }


評估指標之定義
--------------

:math:`\delta`-存在性（:math:`\delta`-presence）是一種隱私保護力指標，表示攻擊者利用個體的背景資訊，成功辨識個體存在於去識別化資料表的機率，是否介於指定數值範圍（:math:`\delta_{\min}` 到 :math:`\delta_{\max}` ）之間，該機率稱 :math:`\delta` 值。一旦攻擊者辨識出個體 存在 或 不存在 於去識別化資料表，可藉由去識別化資料表推敲出個體敏感資訊。

:math:`\delta` 值數值分佈在 0 到 1 之間。當數值越高或越低，攻擊者越容易判斷個體 存在 或 不存在 於去識別化資料表；當數值越接近中間值 0.5，攻擊者越難判斷個體是否存在於去識別化資料表。

因此，指標使用 :math:`\delta_{\min}` 與 :math:`\delta_{\max}` 兩可調配參數限制 :math:`\delta` 值數值範圍。當 :math:`\delta_{\min}` 越大、:math:`\delta_{\max}` 越小，:math:`\delta` 值越接近中間值，此時個體之敏感資訊獲得最大保障。

評估指標之判斷
---------------

參考 [1]_ ，判斷 :math:`\delta`-存在性的方法有三個步驟：

1. 對 原始資料表 做與 去識別化資料表 程度相同之去識別化處理，作為攻擊者已知背景資訊。
2. 參考步驟 1 背景資訊，計算各個體存在於 去識別化資料表 之機率。
3. 判斷步驟 2 計算之機率是否皆介於指定數值範圍之間。

接下來將以下列「原始資料表」與「去識別化資料表」，說明判斷過程。

**原始資料表**

+------+----------+
| 編號 | 郵遞區號 |
+======+==========+
| 1    | 4712     |
+------+----------+
| 2    | 4823     |
+------+----------+
| 3    | 4834     |
+------+----------+

**去識別化資料表**


+------+----------+
| 編號 | 郵遞區號 |
+======+==========+
| 1    | 47**     |
+------+----------+
| 2    | 48**     |
+------+----------+

假設「原始資料表」經過以下去識別化處理，可得到「去識別化資料表」：

1. 將數值範圍為 4700～4799 的「郵遞區號」資料替換成 47** 。
2. 將數值範圍爲 4800～4899 的「郵遞區號」資料替換成 48** 。
3. 移除編號 3 資料列。


若欲判斷「去識別化資料表」是否滿足 :math:`\delta_{\min}` = 1/4 與 :math:`\delta_{\max}` = 1 之 :math:`\delta`-存在性，可以如下進行。

**STEP1: 對 原始資料表 做與 去識別化資料表 程度相同之去識別化處理，作為攻擊者已知背景資訊。**

以「原始資料表」 **編號1資料列** 為例，資料「4712」落在數值範圍 4700～4799，經與 去識別化資料表 程度相同之去識別化處理，「4712」被替換成「47**」。

以「原始資料表」 **編號2資料列** 為例，資料「4823」落在數值範圍 4800～4899，經與 去識別化資料表 程度相同之去識別化處理，「4823」被替換成「48**」。

依以上操作得到下方 攻擊者已知背景資訊表。

**背景資訊表**

+------+----------+
| 編號 | 郵遞區號 |
+======+==========+
| 1    | 47**     |
+------+----------+
| 2    | 48**     |
+------+----------+
| 3    | 48**     |
+------+----------+

**STEP2: 參考步驟 1 背景資訊，計算各個體存在於 去識別化資料表 之機率。**

「背景資訊表」資料列存在於 「去識別化資料表」之機率計算公式如下：



.. math:: 
    \begin{equation}
    \begin{aligned}
    資料列 存在機率  
     = \frac{資料列於「去識別化資料表」之出現次數}{資料列於「背景資訊表」之出現次數} \\ 
    \end{aligned}
    \end{equation}


以「背景資訊表」 **編號 1 資料列** 為例，其在 「背景資訊表」 出現 **一次**，在「去識別化資料表」也出現 **一次**，因此資料列存在機率為 1。


.. math:: 
    \begin{equation}
    \begin{aligned}
    編號 1 資料列存在機率  
     &= \frac{編號1資料列於「去識別化資料表」之出現次數}{編號1資料列於「背景資訊表」之出現次數} \\
     &= \frac{1}{1} = 1
    \end{aligned}
    \end{equation}

以「背景資訊表」 **編號 2 資料列** 為例，其在 「背景資訊表」 出現 **二次**，在「去識別化資料表」則出現 **一次**，因此資料列存在機率為 1/2。


.. math:: 
    \begin{equation}
    \begin{aligned}
    編號 2 資料列存在機率  
     &= \frac{編號2資料列於「去識別化資料表」之出現次數}{編號2資料列於「背景資訊表」之出現次數} \\
     &= \frac{1}{2}
    \end{aligned}
    \end{equation}

同理，得到 **編號 3 資料列**  存在機率為 1/2。

**STEP3: 判斷步驟 2 計算之機率是否皆介於指定數值範圍之間。**

最後，確認步驟 2 計算之機率是否介於 :math:`\delta_{\min}` 與 :math:`\delta_{\max}` 之間。 在此例中，**編號 1 資料列**  存在機率為 1，**編號 2、3 資料列** 存在機率為 1/2 ，皆介於 1/4 與 1 之間。因此，「去識別化資料表」滿足 :math:`\delta_{\min}` = 1/4 與 :math:`\delta_{\max}` = 1 之 :math:`\delta`-存在性。


參考資料
---------

.. [1] M. E. Nergiz, M. Atzori, and C. Clifton, “Hiding the presence of individuals from shared databases,” Proceedings of the 2007 ACM SIGMOD international conference on Management of data, 2007. 

