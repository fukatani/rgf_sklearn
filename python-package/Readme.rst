|Build Status Travis| |Build Status AppVeyor| |License| |Python Versions| |PyPI Version|

.. [![PyPI Version](https://img.shields.io/pypi/v/rgf_python.svg)](https://pypi.org/project/rgf_python/) # Reserve link for PyPI in case of bugs at fury.io

rgf\_python
===========

The wrapper of machine learning algorithm **Regularized Greedy Forest (RGF)** `[1] <#references>`__ for Python.

Features
--------

**Scikit-learn interface and possibility of usage for multiclass classification problem.**

**rgf\_python** contains both vanilla RGF from the paper `[1] <#references>`__  and FastRGF `[2] <#references>`__ implementations.

Note that FastRGF is developed to be used with large (and sparse) datasets, so on small datasets it often shows poorer performance compared to vanilla RGF.

Original RGF implementations are available only for regression and binary classification, but **rgf\_python** is also available for **multiclass classification** by "One-vs-Rest" method.

Examples
--------

.. code:: python

    from sklearn import datasets
    from sklearn.utils.validation import check_random_state
    from sklearn.model_selection import StratifiedKFold, cross_val_score
    from rgf.sklearn import RGFClassifier

    iris = datasets.load_iris()
    rng = check_random_state(0)
    perm = rng.permutation(iris.target.size)
    iris.data = iris.data[perm]
    iris.target = iris.target[perm]

    rgf = RGFClassifier(max_leaf=400,
                        algorithm="RGF_Sib",
                        test_interval=100,
                        verbose=True)

    n_folds = 3

    rgf_scores = cross_val_score(rgf,
                                 iris.data,
                                 iris.target,
                                 cv=StratifiedKFold(n_folds))

    rgf_score = sum(rgf_scores)/n_folds
    print('RGF Classfier score: {0:.5f}'.format(rgf_score))

More examples of using RGF estimators could be found `here <https://github.com/RGF-team/rgf/tree/master/python-package/examples/RGF>`__.

Examples of using FastRGF estimators could be found `here <https://github.com/RGF-team/rgf/tree/master/python-package/examples/FastRGF>`__.

Software Requirements
---------------------

-  Python (2.7 or >= 3.4)
-  scikit-learn (>= 0.18)

Installation
------------

From `PyPI <https://pypi.org/project/rgf_python>`__ using ``pip``:

::

    pip install rgf_python

or from `GitHub <https://github.com/RGF-team/rgf>`__:

::

    git clone https://github.com/RGF-team/rgf.git
    cd rgf/python-package
    python setup.py install

MacOS users, **rgf\_python** after the ``3.1.0`` version is built with **g++-8** and cannot be launched on systems with **g++-7** and earlier. You should update your **g++** compiler if you don't want to build from sources or install **rgf\_python** ``3.1.0`` from PyPI which is the last version built with **g++-7**.

If you have any problems while installing by methods listed above, you should *build RGF and FastRGF executable files from binaries on your own and place compiled executable files* into directory which is included in environmental variable **'PATH'** or into directory with installed package. Alternatively, you may specify actual locations of executable files and directory for placing temp files by corresponding flags in configuration file ``.rgfrc``, which you should create into your home directory. The default values are platform dependent: for Windows ``exe_location=$HOME/rgf.exe``, ``fastrgf_location=$HOME``, ``temp_location=$HOME/temp/rgf`` and for others ``exe_location=$HOME/rgf``, ``fastrgf_location=$HOME``, ``temp_location=/tmp/rgf``. Here is the example of ``.rgfrc`` file:

::

    exe_location=C:/Program Files/RGF/bin/rgf.exe
    fastrgf_location=C:/Program Files/FastRGF/bin
    temp_location=C:/Program Files/RGF/temp

Note that while ``exe_location`` should point to a concrete RGF executable **file**, ``fastrgf_location`` should point to a **folder** in which ``forest_train.exe`` and ``forest_predict.exe`` FastRGF executable files are located.

Also, you may directly specify installation without automatic compilation:

::

    pip install rgf_python --install-option=--nocompilation

or

::

    git clone https://github.com/RGF-team/rgf.git
    cd rgf/python-package
    python setup.py install --nocompilation

``sudo`` (or administrator privileges in Windows) may be needed to perform commands.

Detailed guides how you can build executable files of RGF and FastRGF from source files could be found in their folders `here <https://github.com/RGF-team/rgf/tree/master/RGF#3-creating-the-executable>`__ and `here <https://github.com/RGF-team/rgf/tree/master/python-package#fastrgf-compilation>`__ respectively.

FastRGF Compilation
'''''''''''''''''''

Note that compilation only with g++-5 and newer versions is possible. Other compilers are unsupported and older versions produce corrupted files.

Windows
~~~~~~~

CMake and MinGW-w64
^^^^^^^^^^^^^^^^^^^

On Windows compilation only with `MinGW-w64 <https://mingw-w64.org/doku.php>`__ is supported because only this version provides POSIX threads.

::

    cd rgf/python-package/include/fast_rgf
    mkdir build
    cd build
    cmake .. -G "MinGW Makefiles"
    mingw32-make 
    mingw32-make install

\*nix
~~~~~

CMake
^^^^^

::

    cd rgf/python-package/include/fast_rgf
    mkdir build
    cd build
    cmake ..
    make 
    make install

Docker image
''''''''''''

We provide `docker image <https://github.com/RGF-team/rgf/blob/master/python-package/docker/Dockerfile>`__ with installed **rgf\_python**.

::

    # Run docker image
    docker run -it fukatani/rgf_python /bin/bash
    # Run RGF example
    python ./rgf/python-package/examples/RGF/comparison_RGF_and_RF_regressors_on_boston_dataset.py
    # Run FastRGF example
    python ./rgf/python-package/examples/FastRGF/FastRGF_classifier_on_iris_dataset.py

Tuning Hyper-parameters
-----------------------

RGF
'''

You can tune hyper-parameters as follows.

-  *max\_leaf*: Appropriate values are data-dependent and usually varied from 1000 to 10000.
-  *test\_interval*: For efficiency, it must be either multiple or divisor of 100 (default value of the optimization interval).
-  *algorithm*: You can select "RGF", "RGF Opt" or "RGF Sib".
-  *loss*: You can select "LS", "Log", "Expo" or "Abs".
-  *reg\_depth*: Must be no smaller than 1. Meant for being used with *algorithm* = "RGF Opt" or "RGF Sib".
-  *l2*: Either 1, 0.1, or 0.01 often produces good results though with exponential loss (*loss* = "Expo") and logistic loss (*loss* = "Log"), some data requires smaller values such as 1e-10 or 1e-20.
-  *sl2*: Default value is equal to *l2*. On some data, *l2*/100 works well.
-  *normalize*: If turned on, training targets are normalized so that the average becomes zero.
-  *min\_samples\_leaf*: Smaller values may slow down training. Too large values may degrade model accuracy.
-  *n\_iter*: Number of iterations of coordinate descent to optimize weights.
-  *n\_tree\_search*: Number of trees to be searched for the nodes to split. The most recently grown trees are searched first.
-  *opt\_interval*: Weight optimization interval in terms of the number of leaf nodes.
-  *learning\_rate*: Step size of Newton updates used in coordinate descent to optimize weights.

Detailed instruction of tuning hyper-parameters is `here <https://github.com/RGF-team/rgf/blob/master/RGF/rgf-guide.pdf>`__.

FastRGF
'''''''

-  *n\_estimators*: Typical range is [100, 10000], and a typical value is 1000.
-  *max\_depth*: Controls the tree depth.
-  *max\_leaf*: Controls the tree size.
-  *tree\_gain\_ratio*: Controls when to start a new tree.
-  *min\_samples\_leaf*: Controls the tree growth process.
-  *loss*: You can select "LS", "MODLS" or "LOGISTIC".
-  *l1*: Typical range is [0, 1000], and a large value induces sparsity.
-  *l2*: Use a relatively large value such as 1000 or 10000. The larger value is, the larger *n\_estimators* you need to use: the resulting accuracy is often better with a longer training time.
-  *opt\_algorithm*: You can select "rgf" or "epsilon-greedy".
-  *learning\_rate*: Step size of epsilon-greedy boosting. Meant for being used with *opt\_algorithm* = "epsilon-greedy".
-  *max\_bin*: Typical range for dense data is [10, 65000] and for sparse data is [10, 250].
-  *min\_child\_weight*: Controls the process of discretization (creating bins).
-  *data\_l2*: Controls the degree of L2 regularization for discretization (creating bins).
-  *sparse\_max\_features*: Typical range is [1000, 10000000]. Meant for being used with sparse data.
-  *sparse\_min\_occurences*: Controls which feature will be selected. Meant for being used with sparse data.

Using at Kaggle Kernels
-----------------------

Kaggle Kernels support **rgf\_python**. Please see `this page <https://www.kaggle.com/fukatani/d/uciml/iris/classification-by-regularized-greedy-forest>`__.

Troubleshooting
---------------

If you meet any error, please try to run `test_rgf_python.py <https://github.com/RGF-team/rgf/blob/master/python-package/tests/test_rgf_python.py>`__ to confirm successful package installation.

Then feel free to `open new issue <https://github.com/RGF-team/rgf/issues/new>`__.

Known Issues
''''''''''''

* FastRGF crashes if training dataset is too small (#data < 28). (`rgf#92 <https://github.com/RGF-team/rgf/issues/92>`__)

* **rgf\_python** does not provide any built-in method to calculate feature importances. (`rgf#109 <https://github.com/RGF-team/rgf/issues/109>`__)

FAQ
'''

* Q: Temporary files use too much space on my hard drive (Kaggle Kernels disc space is exhausted while fitting **rgf\_python** model).
   
  A: Please see `rgf#75 <https://github.com/RGF-team/rgf/issues/75>`__.

* Q: GridSearchCV/RandomizedSearchCV/RFECV or other scikit-learn tool with ``n_jobs`` parameter hangs/freezes/crashes when runs with **rgf\_python** estimator.

  A: This is a known general problem of multiprocessing in Python. You should set ``n_jobs=1`` parameter of either estimator or scikit-learn tool.

License
-------

**rgf\_python** is distributed under the MIT license. Please read file `LICENSE <https://github.com/RGF-team/rgf/blob/master/python-package/LICENSE>`__ for more information.

Many thanks to Rie Johnson and Tong Zhang (the authors of RGF).

Other
-----

Shamelessly, some part of the implementation is based on the following `code <https://github.com/MLWave/RGF-sklearn>`__. Thanks!

References
----------

[1] `Rie Johnson and Tong Zhang, Learning Nonlinear Functions Using Regularized Greedy Forest <https://arxiv.org/abs/1109.0887>`__

[2] `Tong Zhang, FastRGF: Multi-core Implementation of Regularized Greedy Forest <https://github.com/baidu/fast_rgf>`__

.. |Build Status Travis| image:: https://travis-ci.org/RGF-team/rgf.svg?branch=master
   :target: https://travis-ci.org/RGF-team/rgf
.. |Build Status AppVeyor| image:: https://ci.appveyor.com/api/projects/status/u3612bfh9pmela42/branch/master?svg=true
   :target: https://ci.appveyor.com/project/RGF-team/rgf
.. |License| image:: https://img.shields.io/badge/license-MIT-blue.svg
   :target: https://github.com/RGF-team/rgf/blob/master/python-package/LICENSE
.. |Python Versions| image:: https://img.shields.io/pypi/pyversions/rgf_python.svg
   :target: https://pypi.org/project/rgf_python/
.. |PyPI Version| image:: https://badge.fury.io/py/rgf_python.svg
   :target: https://badge.fury.io/py/rgf_python