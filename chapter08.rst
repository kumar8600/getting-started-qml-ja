.. -*- coding: utf-8 -*-

テキストエディタの実行
======================

テキストエディタを実行する前に、ファイルダイアログのC++プラグインをビルドする必要があります。それをビルドしたら、ディレクトリ ``filedialog`` に入って、 ``qmake`` を実行し、 ``make`` または ``nmake`` をプラットフォームに合わせて用いてコンパイルしてください。

テキストエディタを `qmlscene`_ を、インポートディレクトリを引数として渡してQMLエンジンに私達のファイルダイアログプラグインのモジュールをどこから探せばいいか分からせて、実行します。:

.. code-block:: bash

    qmlscene -I ./imports texteditor.qml

完全なソースコードはディレクトリ ``examples/quick/tutorials/gettingStartedQml`` にあります。

.. _`qmlscene`: http://qt-project.org/doc/qt-5/qtquick-qmlscene.html
