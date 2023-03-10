+++++++++++++++++++++++++++++++++++++++
資料非均勻熵評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估經去識別化處理後的表格資料之非均勻熵（Non-Uniform Entropy）。更多有關非均勻熵之說明，詳見 `此處 <#id4>`_ 。

我們以 ``data/adult.csv`` 作爲未處理之原始資料、 ``data/adult_anonymized.csv`` 作爲經去識別化處理之結果資料、 ``data/adult_hierarchy`` 目錄作爲資料儲存格定義，展示如何透過 PETWorks-framework 框架計算此指標。

在以下程式碼中，我們透過 API ``PETValidation(original, anonymized, tech, dataHierarchy)``，以上述資料與 “Non-Uniform Entropy” 字串作爲參數，計算資料之非均勻熵。

再來，我們透過 API ``report(result, format)``，以上述評估結果與 “json” 字串作爲參數，將評估結果以 JSON 格式印出。

範例程式碼: nonUniformEntropy.py
----------------------------------

.. code-block:: python

    from PETWorks import PETValidation, report

    originalData = "data/adult.csv"
    anonymizedData = "data/adult_anonymized.csv"
    dataHierarchy = "data/adult_hierarchy"

    result = PETValidation(
        originalData, anonymizedData, "Non-Uniform Entropy", dataHierarchy=dataHierarchy
    )
    report(result, "json")

輸出結果
--------

.. code-block:: bash

    $ python nonUniformEntropy.py
    {
        "Non-Uniform Entropy": 0.6691909578638351
    }


評估指標之定義
--------------

非均勻熵（Non-Uniform Entropy）是一種指標，用來表示資料經過概化處理（Generalization）後，剩餘資訊量的多寡。其中概化處理是一種去識別化方法，是將資料替換成較廣義、籠統的值，例如將年份「1977」替換成「197*」。

而非均勻熵的數值介於 0 與 1 之間，數值越大，代表經概化處理後剩餘的資訊量越多。


舉例來說，一份未經概化處理的資料，其非均勻熵為 1 。假設一份紀錄姓名「王小明」的戶政資料，經概化處理成紀錄「王〇明」之資料，相較於紀錄「王〇〇」之資料，前者非均勻熵較高。

評估指標之計算
--------------

計算非均勻熵的方法有三個步驟：

1. 計算每個儲存格經過概化處理後的資訊損失量。
2. 計算資料表整體的資訊損失量。
3. 以 1 減去整體資訊損失量之正規化結果，得之。

接下來將以下列 「原始資料表」與「去識別化資料表」，說明計算過程。

**原始資料表**

+-----------+-----------+
|   編號    |  出生年   |
+===========+===========+
|          1| 1970      |
+-----------+-----------+
|          2| 1981      |
+-----------+-----------+
|          3| 1983      |
+-----------+-----------+
|          4| 1988      |
+-----------+-----------+

**去識別化資料表**

+-----------+-----------+
|   編號    |  出生年   |
+===========+===========+
|          1| 197*      |
+-----------+-----------+
|          2| 198*      |
+-----------+-----------+
|          3| 198*      |
+-----------+-----------+
|          4| 198*      |
+-----------+-----------+

假設「原始資料表」經過以下概化處理可得到「去識別化資料表」：

1. 將數值範圍爲 1970 ~ 1979 的「出生年」資料替換成 197* 。
2. 將數值範圍爲 1980 ~ 1989 的「出生年」資料替換成 198* 。



**STEP1: 計算每個儲存格經過概化處理後的資訊損失量。**

首先，以資料行爲單位，計算每個儲存格經過概化處理後的資訊損失量，公式如下：

.. math:: 
    \begin{equation}
    \begin{aligned}
    資訊損失量 &= -\log_{2}{(概化處理前與概化處理後資料之數值的出現次數比值)}\\ &=  -\log_{2} {(\frac{概化處理前資料之數值在資料行的出現次數}{概化處理後資料之數值在資料行的出現次數})}
    \end{aligned}
    \end{equation}


根據論文定義 [1]_ ，資料越 **獨特**，其所蘊含的資訊量越豐富；資料越 **常見**，其所蘊含的資訊量越稀少。

假設一筆 **獨特資料** ，經過概化處理後變得 **常見** ，表示「概化處理前資料之數值在資料行的出現次數」較低，「概化處理後資料之數值在資料行的出現次數」較高，因此「出現次數比值」較小，資訊損失量較大。

假設一筆 **獨特資料** ，經過概化處理後依然 **獨特** ，表示「概化處理前資料之數值在資料行的出現次數」較低，「概化處理後資料之數值在資料行的出現次數」較低，因此「出現次數比值」較高，資訊損失量較小。

同理， **常見資料** 經過概化處理後變得 **常見** 亦然，表示資訊損失量較小。


以 **編號1** 儲存格為例，數值 1970 在「原始資料表」的「出生年」資料行中出現 1 次，經過概化處理後變成 197* ，在「去識別化資料表」的「出生年」資料行中出現 1 次，故：


.. math:: 
    \begin{equation}
    \begin{aligned}
    「編號1」儲存格的資訊損失量 &= -\log_{2}{(概化處理前與概化處理後資料之數值的出現次數比值)} \\ &= -\log_{2}{(\frac{概化處理前「編號1」資料之數值在資料行的出現次數}{概化處理後「編號1」資料之數值在資料行的出現次數})} \\ &  = -\log_{2}{ (\frac{1}{1})} = 0
    \end{aligned}
    \end{equation}

以 **編號2** 儲存格為例，數值 1981 在「原始資料表」的「出生年」資料行中出現 1 次，經過概化處理後變成 198*，在「去識別化資料表」的「出生年」資料行中出現 3 次，故：


.. math:: 
    \begin{equation}
    \begin{aligned}
    「編號2」儲存格的資訊損失量 &= -\log_{2}{(概化處理前與概化處理後資料之數值的出現次數比值)} \\ &= -\log_{2}{(\frac{概化處理前「編號2」資料之數值在資料行的出現次數}{概化處理後「編號2」資料之數值在資料行的出現次數})} \\ &  = -\log_{2}{ (\frac{1}{3})}
    \end{aligned}
    \end{equation}



同理，**編號3**\ 與 **編號4**\ 儲存格之資訊損失量皆為 :math:`-\log_{2}{ (\frac{1}{3})}` 。


**STEP2: 計算資料表整體的資訊損失量。**

再來，加總所有儲存格的資訊損失量，得到整體資訊損失量： 



.. math:: 
    \begin{equation}
    \begin{aligned}
    整體資訊損失量 = 所有儲存格損失資訊量的總和 = 0-\log_{2}{ (\frac{1}{3})}-\log_{2}{ (\frac{1}{3})} -\log_{2}{ (\frac{1}{3})} \approx 4.7549
    \end{aligned}
    \end{equation}



**STEP3: 以 1 減去整體資訊損失量之正規化結果，得之。**

最後，計算整體資訊損失量之正規化結果，公式如下:


.. math:: 
    \begin{equation}
    \begin{aligned}
    資訊損失量之正規化結果 = \frac{資訊損失量 - 可損失之最小資訊量}{可損失之最大資訊量 - 可損失之最小資訊量} 
    \end{aligned}
    \end{equation}



資料表的可損失之最小資訊量，爲所有儲存格可損失之最小資訊量的加總。而當儲存格具有可損失之最小資訊量時，即代表儲存格之數值，經概化處理後沒有變化，「出現次數比值」為 1 。因此，資料表的可損失之最小資訊量可如下計算：


.. math:: 
    \begin{equation}
    \begin{aligned}
    資料表的可損失之最小資訊量 &= 資料表的儲存格數量 \times  儲存格的可損失之最小資訊量 
    \\ & = 資料表的儲存格數量 \times -\log_{2} {(概化處理前與概化處理後資料之數值的出現次數比值)} 
    \\ &=資料表的儲存格數量 \times -\log_{2}({1})
    \\ & = 資料表的儲存格數量 \times  0 \\ &=0
    \end{aligned}
    \end{equation}





資料表的可損失之最大資訊量，爲所有儲存格可損失之最大資訊量的加總。而當儲存格具有可損失之最大資訊量時，即代表儲存格之數值，原只出現 1 次 ，經概化處理後，與整個資料行之數值相同。

因此，資料表的可損失之最大資訊量可如下計算：



.. math:: 
    \begin{equation}
    \begin{aligned}
    資料表的可損失之最大資訊量  &= 資料表的儲存格數量 \times 儲存格的可損失之最大資訊量 
    \\ &= 資料表的儲存格數量 \times -\log_{2} {(\frac{概化處理前資料之數值在資料行的出現次數}{概化處理後資料之數值在資料行的出現次數})}
    \\&= 資料表的儲存格數量 \times -\log_{2}{(\frac{1}{資料行的儲存格數量})} 
    \end{aligned}
    \end{equation}


以上述情境爲例，其資料表的儲存格數量為 4 ，資料行的儲存格數量也為 4 ，其可損失之最大資訊量即：


.. math:: 
    \begin{equation}
    \begin{aligned}
    資料表的可損失之最大資訊量  &= 資料表的儲存格數量 \times -\log_{2}{(\frac{1}{資料行的儲存格數量})} 
    \\& = 4 \times -\log_{2}{(\frac{1}{4}) = 8}
    \end{aligned}
    \end{equation}



將可損失之最大資訊量與可損失之最小資訊量，代入正規化公式：


.. math:: 
    \begin{equation}
    \begin{aligned}
    資訊損失量之正規化結果 &= \frac{資訊損失量 - 可損失之最小資訊量}{可損失之最大資訊量 - 可損失之最小資訊量} \\ &= \frac{4.7549 - 0}{8 - 0} \approx 0.5944
    \end{aligned}
    \end{equation}




以 1 減去資訊損失量之正規化結果，得到「去識別化資料表」的非均勻熵：


.. math:: 
    \begin{equation}
    \begin{aligned}
     非均勻熵  = 1- 「去識別化資料表」資訊損失量之正規化結果 ＝ 1- 0.5944 = 0.4056
    \end{aligned}
    \end{equation}





參考資料
--------

.. [1] A. Gionis and T. Tassa, “k-Anonymization with Minimal Loss of Information.” IEEE Transactions on Knowledge and Data Engineering, vol. 21, no. 2, pp. 206-219, 2009, doi: 10.1109/tkde.2008.129.
