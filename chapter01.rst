.. -*- coding: utf-8 -*-

ユーザーインターフェースを作るためのQML
=======================================

私達が作ろうとしているのは、読込、保存、いくつかのテキスト操作の実行をする簡単なテキストエディタです。
このガイドは２つの部から成ります。第一部では、アプリケーションのレイアウトと振る舞いの設計をQMLで行います。そして第二部では、ファイルの読み込みと保存をQt C++で実装します。そのために、 `Qtのメタ・オブジェクトシステム`_ を使って、C++の関数を `QMLオブジェクト型`_ のプロパティとして公開します。QMLとQt C++を利用して、私達はアプリケーション・ロジックからインターフェイス・ロジックを分離することが出来るのです。

.. figure:: http://qt-project.org/doc/qt-5/images/qml-texteditor5_editmenu.png

   完全なソースコードは ``examples/quick/tutorials/gettingStartedQml`` ディレクトリにあります。最終的なアプリケーションがどんな感じか見たければ、 :doc:`chapter08` までスキップしてください。

このチュートリアルのC++の部分は、読者がQtのコンパイル手順の基本的な知識を有していることを前提としています。

:チュートリアルの章:
   1. :doc:`chapter02`
   2. :doc:`chapter03`
   3. :doc:`chapter04`
   4. :doc:`chapter05`
   5. :doc:`chapter06`

文法や機能といった、QMLについての情報は、 `The QML Reference`_ に含まれています。

.. _`Qtのメタ・オブジェクトシステム`: http://qt-project.org/doc/qt-5/metaobjects.html
.. _`QMLオブジェクト型`: http://qt-project.org/doc/qt-5/qtqml-typesystem-objecttypes.html
.. _`The QML Reference`: http://qt-project.org/doc/qt-5/qmlreference.html
