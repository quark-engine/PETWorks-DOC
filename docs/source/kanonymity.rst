+++++++++++++++++++++++++++++++++++++++
資料 k-匿名性評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估資料是否滿足 k-匿名性（k-anonymity）。更多 k-匿名性之說明，詳見 `此處 <#id3>`_ 。

我們以 ``data/adult_anonymized.csv`` 作爲欲評估之表格資料，以及 ``attributeTypes`` 作為屬性型態定義，展示如何透過 PETWorks-framework 框架判斷此指標。

在以下程式碼中，我們透過 API ``PETValidation(None, anonymized, tech, attributeTypes, k)`` ，以上述資料、“k-anonymity” 字串、屬性型態定義以及整數 k 作爲參數，判斷資料是否滿足 k-匿名性。其中，屬性型態定義將 "age" 與 "sex" 欄位設定爲個體背景資訊。

再來，我們透過 API ``report(result, format)`` ，以上述評估結果與 “json” 字串作爲參數，將評估結果以 JSON 格式印出。


範例程式碼: k-anonymity.py
---------------------------

.. code-block:: python

    from PETWorks import PETValidation, report
    from PETWorks.attributetypes import QUASI_IDENTIFIER

    anonymizedData = "data/adult_anonymized.csv"

    attributeTypes = {
        "age": QUASI_IDENTIFIER,
        "sex": QUASI_IDENTIFIER,
    }

    result = PETValidation(
            None, anonymizedData, "k-anonymity", attributeTypes=attributeTypes, k=6
    )
    report(result, "json")

輸出結果
--------

.. code-block:: text

    $ python3 k-anonymity.py
    {
        "k": 6,
        "k-anonymity": true
    }


評估指標之定義
--------------

k-匿名性（k-anonymity）是一種指標，表示資料表中的任何一個資料列，其儲存之個體背景資訊皆與至少 :math:`k-1` 個資料列相同。其中，個體背景資訊爲可相互組合，識別出個體的隱私資訊，又稱準識別符（Quasi-identifier）。


而 k 爲可調配之參數，數值介於 1 與無限大之間，數值越大，具相同個體背景資訊的資料列越多，識別出個體的難度越高。


評估指標之判斷
--------------

參考 [1]_ 與 [2]_ ，判斷 k-匿名性的方法有兩個步驟：

1. 對於各資料列，計算具相同個體背景資訊的資料列數量。
2. 遍歷各資料列，確認上述數值皆大於等於 :math:`k-1` 。

接下來將以下列「範例資料表」，說明判斷過程。


**範例資料表**

+-----------+-----------+-----------+
| 編號      |  年齡     |  性別     |
+===========+===========+===========+
| 1         | 50-59     | Male      |
+-----------+-----------+-----------+
| 2         | 60-69     | Female    |
+-----------+-----------+-----------+
| 3         | 60-69     | Female    |
+-----------+-----------+-----------+
| 4         | 50-59     | Male      |
+-----------+-----------+-----------+

假設「範例資料表」之「年齡」與「性別」欄位為個體背景資訊。若欲判斷「範例資料表」是否滿足 :math:`k = 2` 之 k-匿名性，可如下進行。

首先，對於各資料列，計算具相同個體背景資訊的資料列數量。

以 **編號 1 資料列** 爲例，「年齡」爲 50-59、「性別」爲 Male，具相同個體背景資訊的資料列只有 1 個，爲編號 4 資料列。

以 **編號 2 資料列** 爲例，「年齡」爲 60-69、「性別」爲 Female，具相同個體背景資訊的資料列亦只有 1 個，爲編號 3 資料列。

同理， **編號 3 與編號 4 資料列** 具相同個體背景資訊的資料列數量亦各為 1 個。

再來，遍歷各資料列，確認上述數值皆大於等於 :math:`k-1` 。

以 **「範例資料表」** 為例，在 :math:`k = 2` 之情況下，各資料列之上述數值皆大於等於 :math:`k-1 =1` 。因此，「範例資料表」滿足 :math:`k = 2` 之 k-匿名性。


參考資料
--------

.. [1] L. Sweeney, “K-anonymity: A model for protecting privacy,” International Journal of Uncertainty, Fuzziness and Knowledge-Based Systems, vol. 10, no. 05, pp. 557–570, 2002. 

.. [2] P. Samarati, "Protecting respondents identities in microdata release," in IEEE Transactions on Knowledge and Data Engineering, vol. 13, no. 6, pp. 1010-1027, Nov.-Dec. 2001, doi: 10.1109/69.971193.
