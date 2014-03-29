.. -*- coding: utf-8 -*-
===========================================
 Qt Quickプログラミング入門
===========================================

:ライセンス:
   Copyright (C)  2013 Digia Plc and/or its subsidiary(-ies).
   Permission is granted to copy, distribute and/or modify this document
   under the terms of the GNU Free Documentation License, Version 1.3
   or any later version published by the Free Software Foundation;
   with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
   A copy of the license is included in the section entitled "GNU
   Free Documentation License".

:訳: kumar8600
:原文: `Getting Started Programming with Qt Quick | QtDoc 5.2 | Documentation | Qt Project`__

__ http://qt-project.org/doc/qt-5/gettingstartedqml.html

:バージョン: 2014/03/29

宣言型UI言語、 **QML** の世界へようこそ。
この入門ガイドでは、簡単なテキストエディタをQMLを用いて作ります。
このガイドを読めば、あなたはQMLとQt C++を用いた自分のアプリケーションを開発できるようになるはずです。


ユーザーインターフェースを作るためのQML
=======================================

私達が作ろうとしているのは、読込、保存、いくつかのテキスト操作の実行をする簡単なテキストエディタです。
このガイドは２つの部から成ります。第一部では、アプリケーションのレイアウトと振る舞いの設計をQMLで行います。そして第二部では、ファイルの読み込みと保存をQt C++で実装します。そのため、 `Qtのメタ・オブジェクトシステム`_ を使って、C++の関数を `QMLオブジェクト型`_ のプロパティとして公開します。QMLとQt C++を利用して、私達はアプリケーション・ロジックからインターフェイス・ロジックを分離することが出来るのです。

.. _`Qtのメタ・オブジェクトシステム`: http://qt-project.org/doc/qt-5/metaobjects.html
.. _`QMLオブジェクト型`: http://qt-project.org/doc/qt-5/qtqml-typesystem-objecttypes.html

.. figure:: http://qt-project.org/doc/qt-5/images/qml-texteditor5_editmenu.png

   完全なソースコードは ``examples/quick/tutorials/gettingStartedQml`` ディレクトリにあります。最終的なアプリケーションがどんな感じか見たければ、 `テキストエディタの実行`_ までスキップしてください。

このチュートリアルのC++の部分は、読者がQtのコンパイル手順の基本的な知識を有していることを前提としています。

:チュートリアルの章:
   TODO

文法や機能といった、QMLについての情報は、 `The QML Reference`__ に含まれています。

__ http://qt-project.org/doc/qt-5/qmlreference.html


ボタンとメニューの定義
----------------------

基本要素 - ボタン
~~~~~~~~~~~~~~~~~

まずはボタン作りから、テキストエディタ作りを始めます。機能的には、マウスを気にする領域とラベルを、ボタンは持っています。ユーザーがボタンを押した時、ボタンはアクションを実行します。

QMLでは、基本の視覚要素は `Rectangle`_ 型です。 `QMLオブジェクト型`_ は `QMLプロパティ`_ を、それの見た目と位置をコントロールするために持っています。

.. _`Rectangle`: http://qt-project.org/doc/qt-5/qml-qtquick-rectangle.html
.. _`QMLプロパティ`: http://qt-project.org/doc/qt-5/qtqml-syntax-propertybinding.html

::

   import QtQuick 2.0

   Rectangle {
       id: simpleButton
       color: "grey"
       width: 150; height: 75

       Text {
           id: buttonLabel
           anchors.centerIn: parent
           text: "button label"
       }
   }

まず、文 ``import QtQuick 2.0`` は、 `qmlscene`_ に、後で私達が使うQML型をインポートさせられます。
この行は、すべてのQMLファイルになければなりません。
``import`` 文に含まれるQtモジュールのバージョンに注意してください。

.. _`qmlscene`: http://qt-project.org/doc/qt-5/qtquick-qmlscene.html

この簡単な矩形は、 ``id`` プロパティに束縛された一意な識別子、 ``simpleButton`` を持っています。
この ``Rectangle`` オブジェクトのプロパティは、プロパティ、続いてコロン、そして値をリストすることで束縛されます。
サンプルコードでは、灰色が ``Rectangle`` の ``color`` プロパティに束縛されています。同様に、矩形の横幅と縦幅も束縛しています。

`Text`_ 型は、編集不可のテキスト領域です。私達はこのオブジェクトに ``buttonLabel`` と名づけました。
そのテキスト領域の中身の文字列をセットするため、私達は ``text`` プロパティに値を束縛しています。
それは ``Rectangle`` の範囲内に含まれ、そして中央に配置するために、 ``Text`` オブジェクトのアンカーを、親である ``simpleButton`` に割り当てています。アンカーは他の要素のアンカーへ束縛することが出来るので、レイアウトの割り当てがより簡単に出来ます。

.. _`Text`: http://qt-project.org/doc/qt-5/qml-qtquick-text.html

このコードを ``SimpleButton.qml`` として保存しましょう。 ``qmlscene`` をこのファイルを引数として渡して実行すると、灰色の矩形とテキストラベルが表示されるでしょう。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor1_simplebutton.png

ボタンクリック機能を実装するには、QMLのイベント・ハンドリングが使えます。QMLのイベント・ハンドリングは `Qtのシグナル・アンド・スロット`_ 機構ととても似ています。シグナルが発行されると、つながっているスロットが呼ばれます。

.. _`Qtのシグナル・アンド・スロット`: http://qt-project.org/doc/qt-5/signalsandslots.html

::

   Rectangle {
       id: simpleButton
       ...
    
       MouseArea {
           id: buttonMouseArea
    
           // マウスエリアのすべての辺を矩形のアンカーにアンカーする
           anchors.fill: parent
           // onClicked は正しいマウスボタンクリックをハンドルする
           onClicked: console.log(buttonLabel.text + " clicked")
       }
    }

`MouseArea`_ オブジェクトを私達の ``simpleButton`` に入れます。
``MouseArea`` オブジェクトはマウスの動きが検出されるインタラクティブな領域を表します。
私達のボタンの場合、 ``MouseArea`` を親である ``simpleButton`` へとアンカーしています。
文 ``anchors.fill`` は、 ``anchors`` と呼ばれるプロパティのグループの内部の、 ``fill`` と呼ばれる明示的なプロパティへアクセスします。
QMLは、別の要素へアンカー出来る要素によるレイアウト、すなわちanchor-basedレイアウトを使い、強固なレイアウトを作成するのです。

.. _`MouseArea`: http://qt-project.org/doc/qt-5/qml-qtquick-mousearea.html

``MouseArea`` はたくさんのシグナル・ハンドラーを持っており、それらは定義した ``MouseArea`` 境界の内側でマウスが動く間ずっと呼ばれます。その一つが ``onClicked`` で、それは好ましいマウスボタン（デフォルトでは左クリック）がクリックされた時必ず呼ばれます。そして、アクションをonClickedハンドラーに束縛できます。
私達の例では、マウスエリアがクリックされた時必ず、 ``console.log()`` がテキストを出力します。
``console.log()`` はデバッグ目的でテキストを出力するのに便利です。

``SimpleButton.qml`` のコードは画面にボタンを表示して、それがマウスがクリックされた時にテキストを出力するのに十分です。

::

    Rectangle {
       id: button
       ...
    
       property color buttonColor: "lightblue"
       property color onHoverColor: "gold"
       property color borderColor: "white"
    
       signal buttonClick()
    
       onButtonClick: {
           console.log(buttonLabel.text + " clicked")
       }
    
       MouseArea{
           onClicked: buttonClick()
           hoverEnabled: true
           onEntered: parent.border.color = onHoverColor
           onExited:  parent.border.color = borderColor
       }
    
       // 条件演算子を使って、ボタンの色を決定する
       color: buttonMouseArea.pressed ? Qt.darker(buttonColor, 1.5) : buttonColor
    }

完全な機能を持つボタンが、 ``Button.qml`` です。この記事のコード片は楕円についてなど、いくつかのコードが省略されています。それは、前の部分で既に紹介しているか、今のコードの話には関係がないからです。

カスタムプロパティは、 ``property type name`` 文で宣言されます。コードでは、 ``color`` 型の ``buttonColor`` プロパティが宣言され、値 ``"lightblue"`` が束縛されています。 ``buttonColor`` はあとで、ボタンを塗りつぶす色を決定する条件つき命令で使われます。

.. note::
   プロパティ値を ``=`` イコール記号を使って代入できる。加えて、 ``:`` コロン文字を使って束縛できる。

カスタムプロパティによって、 ``Rectangle`` のスコープ外から内部の値にアクセス出来る。
``int``, ``string``, ``real``, ``variant`` と呼ばれる型も含む、そういった基本的な `QML型`_ が存在する。

.. _`QML型`: http://qt-project.org/doc/qt-5/qtqml-typesystem-basictypes.html

