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

:訳: `kumar8600`_
:原文: `Getting Started Programming with Qt Quick | QtDoc 5.2 | Documentation | Qt Project`__
:バージョン: WIP

.. _`kumar8600`: https://twitter.com/kumar8600
__ http://qt-project.org/doc/qt-5/gettingstartedqml.html

宣言型UI言語、 **QML** の世界へようこそ。
この入門ガイドでは、簡単なテキストエディタをQMLを用いて作ります。
このガイドを読めば、あなたはQMLとQt C++を用いた自分のアプリケーションを開発できるようになるはずです。


ユーザーインターフェースを作るためのQML
=======================================

私達が作ろうとしているのは、読込、保存、いくつかのテキスト操作の実行をする簡単なテキストエディタです。
このガイドは２つの部から成ります。第一部では、アプリケーションのレイアウトと振る舞いの設計をQMLで行います。そして第二部では、ファイルの読み込みと保存をQt C++で実装します。そのために、 `Qtのメタ・オブジェクトシステム`_ を使って、C++の関数を `QMLオブジェクト型`_ のプロパティとして公開します。QMLとQt C++を利用して、私達はアプリケーション・ロジックからインターフェイス・ロジックを分離することが出来るのです。

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
======================


基本要素 - ボタン
-----------------

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

完全な機能を持つボタンが、 ``Button.qml`` です。この記事のコード片は楕円についてなど、いくつかのコードが省略されています。それは、今までの節で既に紹介しているか、今のコードの話には関係がないからです。

カスタムプロパティは、 ``property type name`` 文で宣言されます。コードでは、 ``color`` 型の ``buttonColor`` プロパティが宣言され、値 ``"lightblue"`` が束縛されています。 ``buttonColor`` はあとで、ボタンを塗りつぶす色を決定する条件つき命令で使われます。

.. note::
   プロパティ値は ``:`` コロン文字を使って束縛できるほか、 ``=`` イコール記号を使って代入することも出来ます。

カスタムプロパティのおかげで、 ``Rectangle`` のスコープ外から内部の値にアクセス出来ます。
``int``, ``string``, ``real``, ``variant`` と呼ばれる型も含む、そういった基本的な `QML型`_ が存在します。

.. _`QML型`: http://qt-project.org/doc/qt-5/qtqml-typesystem-basictypes.html

シグナル・ハンドラー ``onEntered`` と ``onExited`` に色を束縛することで、ボタンの上をマウスホバーした時はボタンの枠線を黄色に変え、そのマウスエリアから出て行ったときは元の色に戻します。

シグナル ``buttonClick()`` は ``Button.qml`` で、キーワード ``signal`` をシグナル名の前に置くことで宣言されています。
すべてのシグナルは自動的に作られた ``on`` で始まる名前のハンドラーを持ちます。だから、 ``onButtonClick`` は、 ``buttonClick`` のハンドラーです。
``onButtonClick`` は、その後実行するアクションを割り当てられています。
私達のボタンの例では、 ``onClicked`` マウスハンドラは単純にテキストを表示する ``onButtonClick`` を呼び出します。
``onButtonClick`` は ``Button`` のマウスエリアへ簡単にアクセスするため、外側のオブジェクトを有効にします。
例えば、一つよりも多くの ``MouseArea`` の宣言とシグナル ``buttonClick`` を持つ要素で、それぞれの ``MouseArea`` の区別を付けるなら、シグナル・ハンドラーを使うのが良い。

今、私達には基本的なマウスの動きをハンドルするQMLの要素を実装するのに充分な基礎知識があります。
``Rectangle`` の内側に ``Text`` ラベルを入れ、それのプロパティのカスタマイズをし、マウスの動きに応じたふるまいを実装しました。QMLオブジェクトを入れることでQMLオブジェクトを作るという考え方は、テキストエディター・アプリケーションの場合でも繰り返されます。

このボタンは、アクションを実行するための構成として使われなければ使い物になりません。
次の節では、こうしたボタンをいくつか持つメニューを作ります。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor1_button.png


メニューページの作成
--------------------

ここまでは、唯一のQMLファイルの中で、どうやってオブジェクトを作り、ふるまいを割り当てるかについてカバーしました。この節では、どうやってQML型をインポートするか、どうやって作成したコンポーネントを他のコンポーネントから再利用するかについてカバーします。

メニューはリストの内容を表示し、各要素はアクションを実行する能力を持っています。QMLでは、様々な方法でメニューを作れます。まず、それぞれが異なるアクションをいずれ起こすボタンを含んでいるメニューを作ります。メニューのコードは ``FileMenu.qml`` にあります。

FileMenu.qmlより::

   Row {
        anchors.centerIn: parent
        spacing: parent.width / 6

        Button {
            id: loadButton
            buttonColor: "lightgrey"
            label: "Load"
        }
        Button {
            buttonColor: "grey"
            id: saveButton
            label: "Save"
        }
        Button {
            id: exitButton
            label: "Exit"
            buttonColor: "darkgrey"

            onButtonClick: Qt.quit()
        }
    }

``FileMenu.qml`` では、３つの ``Button`` オブジェクトを宣言しています。子を列に沿って配置するポジショナーである ``Row`` 型の内部で、それらは宣言されています。 ``Button`` の宣言は前の節で使った ``Button.qml`` に属している。新たに作ったボタンで新たなプロパティの束縛を宣言することで、効果的に ``Button.qml`` でセットされたプロパティを上書き出来ます。 ``exitButton`` と呼ばれるボタンはそれがクリックされた時、終了してウィンドウを閉じます。

.. note::
   ``exitButton`` のハンドラー ``onButtonClick`` に加え、 ``Button.qml`` にあるシグナル・ハンドラー ``onButtonClick`` も呼び出されます。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor1_filemenu.png

``Row`` は ``Rectangle`` の中で定義され、ボタンの列のための矩形のコンテナーを作っている。この付加的な矩形はメニューの内側にボタンの列を作る間接的な方法を作っています。

編集メニューの宣言はこの段階ではよく似ています。そのメニューは ``Copy`` 、 ``Paste`` 、 ``Select All`` ラベルをそれぞれ持つボタンを持ちます。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor1_editmenu.png

前もって作ったコンポーネントのインポートとカスタマイズについての知識を身に付けたので、これから、メニュー・バーを、コンポーネントを組み合わせて作りましょう。コンポーネントとは、複数のメニュー・ページのことで、そのメニュー・ページはそれぞれ、メニューの選択肢としての複数のボタンから成ります。まずはそれらを作ります。
また、QMLでデータを組み立てる方法も見て行きます。


メニュー・バーの実装
====================

私達のテキストエディター・アプリケーションはメニュー・バーを使ってメニューを表示する方法が必要になります。そのメニュー・バーは異なるメニューを切り替える事ができ、ユーザーは表示するメニューを選ぶことが出来ます。メニュー切り替えのために、ただメニューを列で表示するよりも多くの構造が必要です。QMLはデータを組み立てるため、また組み立てられたデータを表示するため、モデルとビューを使います。


データモデルとビューの使用
--------------------------

QMLは `データモデル`_ を表示する、異なる `データビュー`_ を持っています。私達のメニュー・バーはその名前を表示するヘッダーを含むメニューをリスト表示します。そのメニューのリストは `ObjectModel`_ の内側で宣言されます。 ``ObjectModel`` 型は、 ``Rectangle`` オブジェクトのような、既に表示可能な項目を含んでいます。 `ListModel`_ 型のような他のモデル型は、それらのデータを表示するためのデリゲートを必要とします。

私達は２つの視覚的な項目を ``menuListModel`` の中に宣言しています。 ``FileMenu`` と ``EditMenu`` です。その２つのメニューをカスタマイズし、 `ListView`_ で表示しています。ファイル ``MenuBar.qml`` はQML宣言を含み、そして ``EditMenu.qml`` では、簡単な編集メニューが定義されています。

.. _`データモデル`: http://qt-project.org/doc/qt-5/qtquick-modelviewsdata-modelview.html#qml-data-models
.. _`データビュー`: http://qt-project.org/doc/qt-5/qtquick-modelviewsdata-modelview.html#qml-data-models
.. _`ObjectModel`: http://qt-project.org/doc/qt-5/qml-qtqml-models-objectmodel.html
.. _`ListModel`: http://qt-project.org/doc/qt-5/qml-qtqml-models-listmodel.html
.. _`ListView`: http://qt-project.org/doc/qt-5/qml-qtquick-listview.html

::

    ObjectModel {
        id: menuListModel

        FileMenu {
            width: menuListView.width
            height: menuBar.height
            color: fileColor
        }

        EditMenu {
            color: editColor
            width: menuListView.width
            height: menuBar.height
        }
    }

`ListView`_ 型はデリゲートによりモデルを表示します。そのデリゲートはモデル項目を ``Row`` オブジェクトかグリッドの中に表示することが出来ます。私達の ``menuListModel`` には既に可視項目があるため、私達はデリゲートを宣言する必要がありません。

::

    ListView {
        id: menuListView

        // アンカーが、ウィンドウのアンカーに反応するように設定
        anchors.fill: parent
        anchors.bottom: parent.bottom
        width: parent.width
        height: parent.height

        // model にデータを含ませる
        model: menuListModel

        // メニュー切り替えの動きを制御
        snapMode: ListView.SnapOneItem
        orientation: ListView.Horizontal
        boundsBehavior: Flickable.StopAtBounds
        flickDeceleration: 5000
        highlightFollowsCurrentItem: true
        highlightMoveDuration: 240
        highlightRangeMode: ListView.StrictlyEnforceRange
    }

加えて、 ``ListView`` は、マウスドラッグや他のジェスチャーに応じる `Flickable`_ から継承されています。上のコードの最後の部分で、 ``Flickable`` のプロパティを、期待通りのフリックの動きに成るように設定しています。特に、プロパティ ``highlightMoveDuration`` はフリックによる遷移の持続時間を変化させます。 ``highlightMoveDuration`` の値がより高ければ、メニュー切り替えはよりゆっくりになります。

``ListView`` はモデル項目を添字で管理し、そしてモデルの各視覚項目へは宣言順に付けられた添字でアクセス可能です。 ``currentIndex`` を変更すれば、 ``ListView`` でハイライトされている項目を効果的に変えることができます。私達のメニュー・バーのヘッダーはこの効果の良い例です。列に２つのボタンが有り、どちらもクリックされた時、現在のメニューを変更します。 ``fileButton`` はクリックされた時、現在のメニューをファイルメニューへと変更し、その添字は、それが ``menuListModel`` の中で最初に宣言されたので、 ``0`` です。同様に、 ``editButton`` はクリックされた時、現在のメニューを ``EditMenu`` へと変更します。

.. _`Flickable`: http://qt-project.org/doc/qt-5/qml-qtquick-flickable.html

矩形 ``labelList`` は値が、それがメニュー・バーの前に表示されると示す ``1`` である ``z`` を持っています。より ``z`` 値が高い項目は、 ``z`` 値がより低い項目よりも前に表示されます。デフォルトの ``z`` の値は ``0`` です。

::

    Rectangle {
        id: labelList
        ...
        z: 1

        Row {
            anchors.centerIn: parent
            spacing: 40

            Button {
                label: "File"
                id: fileButton
                ...
                onButtonClick: menuListView.currentIndex = 0
            }

            Button {
                id: editButton
                label: "Edit"
                ...
                onButtonClick: menuListView.currentIndex = 1
            }
        }
    }

私達が今作ったメニュー・バーは、フリックするか、または上部にあるメニュー名をクリックするかで、メニューにアクセスすることが出来ます。直感的で、応答性がある感じのメニュー画面切り替えが出来ました。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor2_menubar.png


テキストエディタの構築
======================


TextAreaの宣言
--------------

編集できるテキスト領域のないテキストエディタなんて、テキストエディタとは呼べません。QMLの `TextEdit`_ 型は、複数行の編集できるテキスト領域を可能にします。 ``TextEdit`` は、直接ユーザーにテキストを編集することを許さない `Text`_ 型とは異なります。

.. _`TextEdit`: http://qt-project.org/doc/qt-5/qml-qtquick-textedit.html

::

    TextEdit {
        id: textEditor
        anchors.fill: parent
        width: parent.width
        height: parent.height
        color: "midnightblue"
        focus: true

        wrapMode: TextEdit.Wrap

        onCursorRectangleChanged: flickArea.ensureVisible(cursorRectangle)
    }

