+++++++++++++++++++++++++++++++++++++++
Average Equivalence Class Size
+++++++++++++++++++++++++++++++++++++++

The following code snippet evaluate the average equivalence class size [1]_.

We use ``data/adult.csv`` as the original data and ``data/adult_anonymized.csv`` as the anonymized data to demonstrate how to evaluate this indicator through PETWorks-framework.

In the following code snippet, we use the API ``PETValidation(original, anonymized, tech)`` with the data and the string “AECS” as parameters to evaluate the ambiguity.

Then, we use the API ``report(result, format)`` with the evaluation result and the string "json" as parameters to print the evaluation result in JSON format.

Example: aecs.py
------------------------

.. code-block:: python

    from PETWorks import PETValidation, report

    originalData = "data/adult.csv"
    anonymizedData = "data/adult_anonymized.csv"

    result = PETValidation(
        originalData, anonymizedData, "AECS"
        )
    report(result, "json")


Execution Result
------------------

.. code-block:: text

    $ python aecs.py
    {
        "AECS": 0.9992930131052006
    }


Reference
-----------

.. [1] K. LeFevre, D. J. DeWitt and R. Ramakrishnan, "Mondrian Multidimensional K-Anonymity," 22nd International Conference on Data Engineering (ICDE'06), Atlanta, GA, USA, 2006, pp. 25-25, doi: 10.1109/ICDE.2006.101. 
