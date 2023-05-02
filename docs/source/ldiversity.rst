+++++++++++++++++++++++++++++++++++++++
資料 l-多樣性評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估資料是否滿足 :math:`l`-多樣性（:math:`l`-diversity）。更多 :math:`l`-多樣性之說明，詳見 `此處 <#id4>`_ 。

我們以 ``data/inpatient_anonymized.csv`` 作為去識別化資料表，以及 ``attributeTypes`` 作為屬性型態定義，展示如何透過 PETWorks-framework 框架判斷此指標。

在以下程式碼中，我們透過 API ``PETValidation(None, anonymized, "l-diversity", attributeTypes, l)`` ，以上述資料、“l-diversity” 字串與 :math:`l` 作爲參數，判斷資料是否滿足 :math:`l`-多樣性。

再來，我們透過 API ``report(result, format)`` ，以上述評估結果與 “json” 字串作爲參數，將評估結果以 JSON 格式印出。

範例程式碼: l-diversity.py
--------------------------

.. code-block:: python

    from PETWorks import PETValidation, report
    from PETWorks.attributetypes import SENSITIVE_ATTRIBUTE, QUASI_IDENTIFIER

    anonymized = "data/inpatient_anonymized.csv"

    attributeTypes = {
        "zipcode": QUASI_IDENTIFIER,
        "age": QUASI_IDENTIFIER,
        "nationality": QUASI_IDENTIFIER,
        "condition": SENSITIVE_ATTRIBUTE
    }

    result = PETValidation(
        None, anonymized, "l-diversity", attributeTypes=attributeTypes, l=3
    )
    report(result, "json")

輸出結果
--------

.. code-block:: text
    
    $ python3 l-diversity.py
    {
        "l": 3,
        "fulfill l-diversity": true
    }

評估指標之定義
--------------

:math:`l`-多樣性（:math:`l`-diversity）是一種隱私保護力指標，表示攻擊者利用個體背景資訊，查詢到的敏感資訊數值是否至少有 :math:`l` 種。一旦攻擊者查詢到的數值種類過少，可藉由資料表推敲出個體的敏感資訊數值。

而 :math:`l` 為可調配參數，數值分佈在 1 到無限大之間。數值越高，攻擊者查詢到的數值種類越多，猜中個體敏感資訊數值的機率越低，敏感資訊獲得的保障越大。


評估指標之判斷
---------------

參考 [1]_ 與 [2]_ ，判斷 :math:`l`-多樣性的方法有兩個步驟：

1. 將具有相同個體背景資訊之資料列分為同組，盤點其擁有的敏感資訊數值。
2. 確認各組擁有的敏感資訊數值是否至少有 :math:`l` 種。

接下來將以下列「範例資料表」說明判斷過程。


**範例資料表**

+------+----------+-----------------+
| 編號 | 出生年   | 疾病            |
+======+==========+=================+
| 1    | 197*     | cancer          |
+------+----------+-----------------+
| 2    | 197*     | viral infection |
+------+----------+-----------------+
| 3    | 198*     | viral infection |
+------+----------+-----------------+
| 4    | 198*     | heart disease   |
+------+----------+-----------------+
| 5    | 198*     | cancer          |
+------+----------+-----------------+


假設「範例資料表」之「出生年」為個體背景資訊，「疾病」為敏感資訊。若欲判斷「範例資料表」是否滿足 :math:`l = 2` 之 :math:`l`-多樣性，可以如下進行。

**STEP 1：將具有相同個體背景資訊之資料列分為同組，盤點其擁有的敏感資訊數值。** 

以 「出生年」資料「197*」為例，將具有此個體背景資訊之 **編號 1、2 資料列** 分為同組，盤點其「疾病」資料，得「cancer」與「viral infection」 **2** 種數值。

以 「出生年」資料「198*」為例，將具有此個體背景資訊之 **編號 3、4、5 資料列** 分為同組，盤點其「疾病」資料，得「viral infection」、「heart disease」與「cancer」 **3** 種數值。

**STEP 2：確認各組擁有的敏感資訊數值是否至少有 l 種。**

最後，確認步驟 1 盤點之敏感資訊數值是否至少有 :math:`l` 種。在此例中， **編號 1、2 資料列** 的「疾病」資料有 **2** 種， **編號 3、4、5 資料列** 的「疾病」資料有 **3** 種，皆至少有 2 種。因此，「範例資料表」滿足 :math:`l = 2` 之 :math:`l`-多樣性。

參考資料
---------

.. [1] A. Machanavajjhala, J. Gehrke, D. Kifer, and M. Venkitasubramaniam, L-diversity: privacy beyond k-anonymity. 2006. doi: 10.1109/icde.2006.1.

.. [2] S. M. Stammler, S. Katzenbeisser, and K. Hamacher, “Correcting Finite Sampling Issues in Entropy l-diversity,” in Lecture Notes in Computer Science, Springer Science+Business Media, 2016. doi: 10.1007/978-3-319-45381-1_11.
