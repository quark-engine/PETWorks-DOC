+++++++++++++++++++++++++++++++++++++++
資料平均等價類大小評估指標
+++++++++++++++++++++++++++++++++++++++

以下程式碼旨在評估經去識別化處理後的表格資料之平均等價類大小（Average Equivalence Class Size，AECS）。更多有關平均等價類大小之說明，詳見 `此處 <#id4>`_ 。

我們以 ``data/adult.csv`` 作爲未處理之原始資料與 ``data/adult_anonymized.csv`` 作爲經去識別化處理之資料，展示如何透過 PETWorks-framework 框架計算此指標。

在以下程式碼中，我們透過 API ``PETValidation(original, anonymized, tech)``，以上述資料與 “AECS” 字串作爲參數，計算資料之平均等價類大小。

再來，我們透過 API ``report(result, format)``，以上述評估結果與 “json” 字串作爲參數，將評估結果以 JSON 格式印出。


範例程式碼: aecs.py
------------------------

.. code-block:: python

    from PETWorks import PETValidation, report

    originalData = "data/adult.csv"
    anonymizedData = "data/adult_anonymized.csv"

    result = PETValidation(
        originalData, anonymizedData, "AECS"
        )
    report(result, "json")


輸出結果
--------

.. code-block:: text

    $ python aecs.py
    {
        "AECS": 0.9992930131052006
    }



評估指標之定義
--------------

平均等價類大小（Average Equivalence Class Size）是一種指標，用來表示資料表經去識別化後，資料列的平均重複次數。其中，若兩個資料列在資料表中相同，此兩個資料列屬於同一個等價類；若一個資料列在資料表中獨一無二，則這個資料列獨自構成一個等價類。

而平均等價類大小的數值介於 1 與「資料列數量」之間，數值越大，資料表內的資料列越趨近相同。若數值為 1，代表所有資料列皆不同；若數值為「資料列數量」，代表所有資料列皆相同。

不過，為了方便與原始資料比較，PETWorks 整合之工具 `ARX <https://arx.deidentifier.org/>`_ 會在計算結果後另做額外處理，詳見 `此處 <#id8>`_ 。




評估指標之計算
--------------
參考 [1]_，計算平均等價類大小的方法只有一個步驟：

1. 計算資料列數量除以等價類數量，得到平均等價類大小。

接下來將以下列 「原始資料表」與「去識別化資料表」，說明計算過程。


**原始資料表**

+-----------+-----------+
| 年齡      | 性別      |
+===========+===========+
| 53        | Male      |
+-----------+-----------+
| 65        | Female    |
+-----------+-----------+
| 53        | Female    |
+-----------+-----------+

**去識別化資料表**

+-----------+-----------+
| 年齡      | 性別      |
+===========+===========+
| 53        | \*        |
+-----------+-----------+
| 65        | Female    |
+-----------+-----------+
| 53        | \*        |
+-----------+-----------+

假設「原始資料表」經以下去識別化處理可得到「去識別化資料表」：

1.  將第一、三列的「性別」資料遮蔽，使數值 Male 與 Female 變成 * 。

依以下公式計算，可得到「去識別化資料表」之平均等價類大小：

.. math:: 
    平均等價類大小= 平均ㄧ個等價類的資料列數量 = \frac{資料列數量}{等價類數量}

其中，「去識別化資料表」有 **3** 個資料列與 **2** 個等價類，因此「去識別化資料表」之平均等價類大小為 3/2 。

.. math:: 
    平均等價類大小= 平均ㄧ個等價類的資料列數量 ＝\frac{資料列數量}{等價類數量} = \frac{3}{2}



評估指標之 ARX 工具計算方式
---------------------------

然而，在 ARX 工具中，為了方便與原始資料比較，除依據前述步驟計算平均等價類大小，也額外將其正規化再以 1 減去，得到 ARX 工具定義之平均等價類大小。

這麼做可以讓數值範圍介於 0 與 1 之間，並且數值越大表示資料列重複次數越低，與原始資料差異越小；數值越小表示資料列重複次數越高，與原始資料差異越大。


首先，計算平均等價類大小之正規化結果，公式如下：

 
.. math:: 
    \begin{equation}
    \begin{aligned}
    「平均等價類大小」正規化結果  
     = \frac{平均等價類大小 - 平均等價類大小之最小值}{平均等價類大小之最大值 - 平均等價類大小之最小值} \\ 
    \end{aligned}
    \end{equation}



**平均等價類大小之最小值**，發生在資料表與原始資料表相同時，這是因為 ARX 工具將原始資料表設爲比較基準，此時資料表內等價類數量與原始資料表相同，因此可以如下計算：

 
.. math:: 
    \begin{equation}
    \begin{aligned} 
    平均等價類大小之最小值 &= \frac{資料列數量}{等價類數量} 
    =\frac{原始資料表之資料列數量}{原始資料表之等價類數量}
    \end{aligned}
    \end{equation}


以上例來說，「原始資料表」各有 3 個資料列與等價類，因此平均等價類大小之最小值為 1：

 
.. math:: 
    \begin{equation}
    \begin{aligned} 
    平均等價類大小之最小值 & 
    =\frac{「原始資料表」資料列數量}{「原始資料表」等價類數量}
    = \frac{3}{3} =1
    \end{aligned}
    \end{equation}



**平均等價類大小之最大值**，發生在資料表內所有資料列皆相同時，此時資料表內只有 1 個等價類，因此可以如下計算：


.. math:: 
    \begin{equation}
    \begin{aligned} 
    平均等價類大小之最大值 &= \frac{資料列數量}{等價類數量} 
    =\frac{資料列數量}{1}
    \end{aligned}
    \end{equation}


以上面情境來說，「去識別化資料表」有 3 個資料列，因此平均等價類大小之最大值為 3：


.. math:: 
    \begin{equation}
    \begin{aligned} 
    平均等價類大小之最大值 & =\frac{資料列數量}{1} = \frac{3}{1} = 3
    \end{aligned}
    \end{equation}


最後，將平均等價類大小之最小值與平均等價類大小之最大值，帶入正規化公式：

 
.. math:: 
    \begin{equation}
    \begin{aligned}
    「平均等價類大小」正規化結果 
    &  = \frac{平均等價類大小 - 平均等價類大小之最小值}{平均等價類大小之最大值 - 平均等價類大小之最小值} \\ & = \cfrac{\frac{3}{2}-1}{3-1} =0.25
    \end{aligned}
    \end{equation}



以 1 減去「平均等價類大小」正規化結果，得到「ARX 工具定義之平均等價類大小」： 


 
.. math:: 
    \begin{equation}
    \begin{aligned}
    ARX工具定義之平均等價類大小 & = 1- 「平均等價類大小」正規化結果  \\
    &   = 1 -  0.25 = 0.75
    \end{aligned}
    \end{equation}





參考資料
--------

.. [1] K. LeFevre, D. J. DeWitt and R. Ramakrishnan, "Mondrian Multidimensional K-Anonymity," 22nd International Conference on Data Engineering (ICDE'06), Atlanta, GA, USA, 2006, pp. 25-25, doi: 10.1109/ICDE.2006.101. 
