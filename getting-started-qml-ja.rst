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

`Text`_ 型は、編集不可のテキストエリアです。私達はこのオブジェクトに ``buttonLabel`` と名づけました。
そのテキストエリアの中身の文字列をセットするため、私達は ``text`` プロパティに値を束縛しています。
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

``MouseArea`` はたくさんのシグナル・ハンドラーを持っており、それらは定義した ``MouseArea`` 境界の内側でマウスが動く間ずっと呼ばれます。その一つが ``onClicked`` で、それは好ましいマウスボタン（デフォルトでは左クリック）がクリックされるたびに呼ばれます。そして、アクションをonClickedハンドラーに束縛できます。
私達の例では、マウスエリアがクリックされるたびに、 ``console.log()`` がテキストを出力します。
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

前もって作ったコンポーネントのインポートとカスタマイズについての知識を身に付けたので、これから、メニューバーを、コンポーネントを組み合わせて作りましょう。コンポーネントとは、複数のメニュー・ページのことで、そのメニュー・ページはそれぞれ、メニューの選択肢としての複数のボタンから成ります。まずはそれらを作ります。
また、QMLでデータを組み立てる方法も見て行きます。


メニューバーの実装
====================

私達のテキストエディター・アプリケーションはメニューバーを使ってメニューを表示する方法が必要になります。そのメニューバーは異なるメニューを切り替える事ができ、ユーザーは表示するメニューを選ぶことが出来ます。メニュー切り替えのために、ただメニューを列で表示するよりも多くの構造が必要です。QMLはデータを組み立てるため、また組み立てられたデータを表示するため、モデルとビューを使います。


データモデルとビューの使用
--------------------------

QMLは `データモデル`_ を表示する、異なる `データビュー`_ を持っています。私達のメニューバーはその名前を表示するヘッダーを含むメニューをリスト表示します。そのメニューのリストは `ObjectModel`_ の内側で宣言されます。 ``ObjectModel`` 型は、 ``Rectangle`` オブジェクトのような、既に表示可能な項目を含んでいます。 `ListModel`_ 型のような他のモデル型は、それらのデータを表示するためのデリゲートを必要とします。

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

``ListView`` はモデル項目を添字で管理し、そしてモデルの各視覚項目へは宣言順に付けられた添字でアクセス可能です。 ``currentIndex`` を変更すれば、 ``ListView`` でハイライトされている項目を効果的に変えることができます。私達のメニューバーのヘッダーはこの効果の良い例です。列に２つのボタンが有り、どちらもクリックされた時、現在のメニューを変更します。 ``fileButton`` はクリックされた時、現在のメニューをファイルメニューへと変更し、その添字は、それが ``menuListModel`` の中で最初に宣言されたので、 ``0`` です。同様に、 ``editButton`` はクリックされた時、現在のメニューを ``EditMenu`` へと変更します。

.. _`Flickable`: http://qt-project.org/doc/qt-5/qml-qtquick-flickable.html

矩形 ``labelList`` は値が、それがメニューバーの前に表示されると示す ``1`` である ``z`` を持っています。より ``z`` 値が高い項目は、 ``z`` 値がより低い項目よりも前に表示されます。デフォルトの ``z`` の値は ``0`` です。

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

私達が今作ったメニューバーは、フリックするか、または上部にあるメニュー名をクリックするかで、メニューにアクセスすることが出来ます。直感的で、応答性がある感じのメニュー画面切り替えが出来ました。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor2_menubar.png


テキストエディタの構築
======================


TextAreaの宣言
--------------

編集できるテキストエリアのないテキストエディタなんて、テキストエディタとは呼べません。QMLの `TextEdit`_ 型は、複数行の編集できるテキストエリアを可能にします。 ``TextEdit`` は、直接ユーザーにテキストを編集することを許さない `Text`_ 型とは異なります。

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

エディタは、フォント ``color`` プロパティを設定され、そして ``wrapMode`` をテキストを折り返すように設定されています。 ``TextEdit`` 領域は、テキストカーソルが可視領域の外にあるならスクロールするフリック可能要素の内側にあります。関数 ``ensureVisible()`` は、カーソル矩形が可視境界の外側に出たかチェックし、適宜テキストエリアを移動します。QMLはスクリプトにJavascriptの構文を使用しており、前述のとおり、JavascriptファイルをQMLにインポートして使うことが出来ます。

::

    function ensureVisible(r) {
        if (contentX >= r.x)
            contentX = r.x;
        else if (contentX + width <= r.x + r.width)
            contentX = r.x + r.width - width;
        if (contentY >= r.y)
            contentY = r.y;
        else if (contentY + height <= r.y + r.height)
            contentY = r.y + r.height - height;
    }


テキストエディタの部品の組み立て
--------------------------------

QMLを使って私達のテキストエディタを作る準備が整いました。テキストエディタは２つの部品を持ちます。先ほど作ったメニューバーと、テキストエリアです。QMLは部品を再利用することができるので、部品のインポートと、必要あらばカスタマイズすることで、私達のコードをより単純にします。私達のテキストエディタはウィンドウを２つに分けます。画面の３分の１はメニューバーに捧げられ、３分の２はテキストエリアを表示します。メニューバーは他のどのオブジェクトよりも前に表示されます。

::

    Rectangle {
        id: screen
        width: 1000
        height: 1000

        // 画面は MenuBar と TextArea へと分割される。
        // そのうち３分の１は MenuBar へ割り当てられる。
        property int partition: height / 3

        MenuBar {
            id: menuBar
            height: partition
            width: parent.width
            z: 1
        }

        TextArea {
            id: textArea
            anchors.bottom: parent.bottom
            y: partition
            color: "white"
            width: parent.width
            height: partition * 2
        }
    }

再利用可能な部品をインポートすることで、私達の ``TextEditor`` コードは大変単純になったようです。そういうわけで、プロパティに定義された振る舞いについて気にすることなく、アプリケーションの主要部を作ることができます。このアプローチを使って、アプリケーションのレイアウトやUI部品は簡単に作られます。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor3_texteditor.png


テキストエディタの装飾
======================


引き出しインターフェースの実装
------------------------------

私達のテキストエディタはシンプルに見えますし、それを装飾する必要があります。QMLを使って、私達のテキストエディタの遷移を宣言したり、アニメーションさせたり出来ます。私達のメニューバーは画面の３分の１を占領しているので、欲しい時だけ姿を見せてくれると良いでしょう。

メニューバーがクリックされたとき伸び縮みする、引き出しインターフェースを追加できます。私達の実装では、マウスクリックに反応する細い矩形を持ちます。 ``drawer`` は、引き出しと同じように、２つの状態を持ちます。状態「引き出しは開いている」と、状態「引き出しは閉じている」です。項目 ``drawer`` は、高さが小さく細長い矩形です。入れ子になった、矢印アイコンを表す `Image`_ オブジェクトが、引き出しの内側の中央に配置されるよう宣言されています。引き出しは、ユーザーにマウスエリアをクリックされるたびに、識別子 ``screen`` でアプリケーションの全体へ、状態を代入します。

.. _`Image`: http://qt-project.org/doc/qt-5/qml-qtquick-image.html

::

    Rectangle {
        id: drawer
        height: 15

        Image {
            id: arrowIcon
            source: "images/arrow.png"
            anchors.horizontalCenter: parent.horizontalCenter
        }

        MouseArea {
            id: drawerMouseArea
            anchors.fill: parent

            onClicked: {
                if (screen.state == "DRAWER_CLOSED")
                    screen.state = "DRAWER_OPEN"
                else if (screen.state == "DRAWER_OPEN")
                    screen.state = "DRAWER_CLOSED"
            }
            ...
        }
    }

状態とは単なる構成の集合であり、それは `State`_ 型で宣言されます。状態のリストはリスト可能で、プロパティ ``states`` に束縛されます。私達のアプリケーションでは、 ``DRAWER_CLOSED`` と ``DRAWER_OPEN`` と名づけられた２つの状態があります。項目の構成は `PropertyChanges`_ オブジェクトで宣言されます。状態 ``DRAWER_OPEN`` の中には、プロパティの変化を受け取る項目が４つあります。一つ目のターゲットは、 ``menuBar`` のプロパティ ``y`` を ``0`` に変更する、です。同様に、 ``textArea`` は状態が ``DRAWER_OPEN`` のとき、より低いところを新たな位置にとります。 ``textArea`` 、 ``drawer`` 、drawerのアイコンは、現在の状態を満たすために、プロパティの変更を受けることになります。

.. _`State`: http://qt-project.org/doc/qt-5/qml-qtquick-state.html
.. _`PropertyChanges`: http://qt-project.org/doc/qt-5/qml-qtquick-propertychanges.html

::

    states:[
        State {
            name: "DRAWER_OPEN"
            PropertyChanges { target: menuBar; y: 0 }
            PropertyChanges { target: textArea; y: partition + drawer.height }
            PropertyChanges { target: drawer; y: partition }
            PropertyChanges { target: arrowIcon; rotation: 180 }
        },
        State {
            name: "DRAWER_CLOSED"
            PropertyChanges { target: menuBar; y: -height; }
            PropertyChanges { target: textArea; y: drawer.height; height: screen.height - drawer.height }
            PropertyChanges { target: drawer; y: 0 }
            PropertyChanges { target: arrowIcon; rotation: 0 }
        }
    ]

状態変化は不意に起こりながらも、スムーズな遷移を必要とします。状態間の遷移は、項目のプロパティ ``transitions`` に、 `Transition`_ 型オブジェクトを束縛して定義されます。私達のテキストエディタは ``DRAWER_OPEN`` か ``DRAWER_CLOSED`` のどちらかの状態へ変化するたびに呼ばれる状態遷移を持ちます。重大なことに、遷移は状態 ``from`` と ``to`` が必要ですが、私達の遷移には、ワイルドカードシンボル ``*`` が、すべての状態変化に遷移を適用すると示すために使えます。

``transitions`` に、プロパティ変化のアニメーションを割り当てられます。私達の ``menuBar`` は位置を ``y: 0`` から ``y: -partition`` へと移し、 `NumberAnimation`_ 型を使って、遷移をアニメーションさせられます。 ``target`` のプロパティを宣言して、一定時間、一定の緩和曲線でアニメーションさせます。緩和曲線はアニメーション速度と補完動作を、状態遷移の間、制御します。私達が選んだ `Easing.OutExpo`_ は、アニメーションの終わりの近くで、ゆっくりになる緩和曲線です。より詳しい情報は、QMLの記事、 `アニメーション`_ を見てください。

.. _`Transition`: http://qt-project.org/doc/qt-5/qml-qtquick-transition.html
.. _`NumberAnimation`: http://qt-project.org/doc/qt-5/qml-qtquick-numberanimation.html
.. _`Easing.OutExpo`: http://qt-project.org/doc/qt-5/qml-qtquick-propertyanimation.html#easing.type-prop
.. _`アニメーション`: http://qt-project.org/doc/qt-5/qtquick-statesanimations-animations.html

::

    transitions: [
        Transition {
            to: "*"
            NumberAnimation { target: textArea; properties: "y, height"; duration: 100; easing.type:Easing.OutExpo }
            NumberAnimation { target: menuBar; properties: "y"; duration: 100; easing.type: Easing.OutExpo }
            NumberAnimation { target: drawer; properties: "y"; duration: 100; easing.type: Easing.OutExpo }
        }
    ]

プロパティの変化をアニメーションさせるもう一つの方法は、 `Behavior`_ 型を宣言することです。遷移は状態変化時にのみ動作し、そして ``Behavior`` は一般的なプロパティ変化のアニメーションを設定できます。テキストエディタでは、矢印が ``NumberAnimation`` を持ち、プロパティ ``rotation`` の変化をアニメーションさせます。

TextEditor.qml より::

    Behavior {
        NumberAnimation { property: "rotation"; easing.type: Easing.OutExpo }
    }

私達の部品と、状態とアニメーションの知識の話に戻りましょう。私達は部品の外観をより良くできます。 ``Button.qml`` では、 ``color`` を追加し、プロパティ ``scale`` をボタンがクリックされた時に変化させることが出来ます。色型は `ColorAnimation`_ を用いてアニメーションさせることが出来、数は `NumberAnimation`_ で出来ます。以下で示している構文 ``on propertyName`` は、唯一のプロパティをターゲットとする時役に立ちます。

.. _`ColorAnimation`: http://qt-project.org/doc/qt-5/qml-qtquick-coloranimation.html
.. _`NumberAnimation`: http://qt-project.org/doc/qt-5/qml-qtquick-numberanimation.html

Button.qml より::

    ...

    color: buttonMouseArea.pressed ? Qt.darker(buttonColor, 1.5) : buttonColor
    Behavior on color { ColorAnimation{ duration: 55 } }

    scale: buttonMouseArea.pressed ? 1.1 : 1.0
    Behavior on scale { NumberAnimation{ duration: 55 } }

加えて、QMLの部品の外観を向上させるために、グラデーションのようなカラーエフェクトや透明度エフェクトが使えます。 `Gradient`_ オブジェクトを宣言すると、プロパティ ``color`` は上書きされます。グラデーションの色は、 `GradientStop`_ 型を使って宣言できます。グラデーションは ``0.0`` から ``1.0`` までの間の比率で位置づけられます。

.. _`Gradient`: http://qt-project.org/doc/qt-5/qml-qtquick-gradient.html
.. _`GradientStop`: http://qt-project.org/doc/qt-5/qml-qtquick-gradientstop.html

MenuBar.qml より::

    gradient: Gradient {
        GradientStop { position: 0.0; color: "#8C8F8C" }
        GradientStop { position: 0.17; color: "#6A6D6A" }
        GradientStop { position: 0.98; color: "#3F3F3F" }
        GradientStop { position: 1.0; color: "#0e1B20" }
    }

このグラデーションはメニューバーで奥行きに似せたものを見せるために使われます。最初の色は ``0.0`` から始まり、最後の色は ``1.0`` にあります。


これからすること
--------------------------

私達はとても単純なテキストエディタのユーザーインターフェイスを組み立てました。今後は、ユーザーインターフェイスが完成している中で、普通のQtとC++を用いてアプリケーションロジックを実装することができます。QMLはプロトタイピングツールとして良く動き、アプリケーションロジックとUIデザインを引き離し分離させるのです。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor4_texteditor.png


Qt C++を用いたQMLの拡張
=======================

テキストエディタのレイアウトが出来たので、今C++でテキストエディタの機能を実装することが出来ます。QMLとC++を共に使うことで、Qtで私達のアプリケーションロジックを作ることが可能となります。C++アプリケーションで `QtのQuickクラス`_ を用いることで、QMLコンテクストを作成でき、そして `QQuickView`_ を用いてQML型を表示する事ができます。代わりに、C++コードを拡張プラグインへとエクスポートして、新たな `identified module`_ としてQMLからアクセス出来るようにすることも出来ます。 `qmlscene`_ でQMLファイルを起動するとき、 `import paths`_ の一つからモジュールが見つかることさえ保証されていれば良いです。私達のアプリケーションでは、後者のアプローチを採ります。こうして、実行可能ファイルを実行するのではなく、QMLファイルを ``qmlscene`` から直接読み込むことが出来るのです。

.. _`QtのQuickクラス`: http://qt-project.org/doc/qt-5/qtqml-cppintegration-topic.html
.. _`QQuickView`: http://qt-project.org/doc/qt-5/qquickview.html
.. _`identified module`: http://qt-project.org/doc/qt-5/qtqml-modules-identifiedmodules.html
.. _`import paths`: http://qt-project.org/doc/qt-5/qtqml-syntax-imports.html#qml-import-path


C++クラスをQMLへ公開
----------------------

QtとC++を用いて、読込と保存を実装します。C++クラスと関数は登録することで、QMLで使うことが出来ます。また、それらはQtプラグインとしてコンパイルされ、QMLモジュールとして公開される必要があります。

私達ののアプリケーションでは、以下の項目を作る必要があります
    1. ディレクトリに関係した操作をハンドルするクラス ``Directory``
    2. ディレクトリの中のファイルを模した、 `QObject`_ であるクラス ``File``
    3. QMLコンテクストにクラスを登録する、プラグインクラス
    4. プラグインをコンパイルする、Qtプロジェクトファイル
    5. 識別子を定義（URIをインポート）し、中身（この場合私達のプラグイン）をQMLモジュールから利用可能にする、 `Module definition qmldir ファイル`_ 

.. _`QObject`: http://qt-project.org/doc/qt-5/qobject.html
.. _`Module definition qmldir ファイル`: http://qt-project.org/doc/qt-5/qtqml-modules-qmldir.html

.. note::
   Qt 5.1から、 `Qt Quick Dialogs`_ モジュールが、ローカルファイルシステムからファイルを選択するのに使えるファイルダイアログの部品を提供しています。説明のために、このチュートリアルでは私達自身で記述します。


Qtプラグインのビルド
--------------------

プラグインをビルドするには、Qtプロジェクトファイルに次のように設定する必要があります。まず必要なソース、ヘッダー、およびQtモジュールを私達のプロジェクトファイルに追加する必要があります。すべてのC++コードとプロジェクトファイルは ``filedialog`` ディレクトリにあります。

filedialog.pro より::

    TEMPLATE = lib
    CONFIG += qt plugin
    QT += qml

    DESTDIR +=  ../imports/FileDialog
    OBJECTS_DIR = tmp
    MOC_DIR = tmp

    TARGET = filedialogplugin

    HEADERS += \
            directory.h \
            file.h \
            dialogPlugin.h

    SOURCES += \
            directory.cpp \
            file.cpp \
            dialogPlugin.cpp

プロジェクトと ``qml`` モジュールをリンクして、 ``plugin`` として構成するため、 ``lib`` テンプレートを用いていることが重要です。私達は、コンパイルしたプラグインを親ディレクトリの ``imports/FileDialog`` に置いています。


クラスをQMLへ登録
-----------------

dialogPlugin.h より::

    #include <QtQml/QQmlExtensionPlugin>

    class DialogPlugin : public QQmlExtensionPlugin
    {
        Q_OBJECT
        Q_PLUGIN_METADATA(IID "org.qt-project.QmlExtensionPlugin.FileDialog")

    public:
        // registerTypes は QQmlExtensionPlugin より継承
        void registerTypes(const char *uri);
    };

マクロ `Q_PLUGIN_METADATA`_ を用いて、プラグインをエクスポートする必要があります。私達の ``dialogPlugin.h`` では、マクロ `Q_OBJECT`_ をクラスの最上部に持っています。その上、プロジェクトファイルに ``qmake`` を実行して、必要なメタ・オブジェクトコードを生成する必要があります。

私達のプラグインクラス ``DialogPlugin`` は、 `QQmlExtensionPlugin`_ のサブクラスです。私達は継承した関数 `registerTypes()`_ を実装する必要があります。

.. _`Q_PLUGIN_METADATA`: http://qt-project.org/doc/qt-5/plugins-howto.html
.. _`Q_OBJECT`: http://qt-project.org/doc/qt-5/qobject.html#Q_OBJECT
.. _`QQmlExtensionPlugin`: http://qt-project.org/doc/qt-5/qqmlextensionplugin.html

DialogPlugin.cpp より::

    #include "dialogPlugin.h"
    #include "directory.h"
    #include "file.h"
    #include <QtQml>

    void DialogPlugin::registerTypes(const char *uri)
    {
        // クラス Directory をQMLに "Directory" 型、バージョン 1.0 として登録
        // @uri FileDialog
        qmlRegisterType<Directory>(uri, 1, 0, "Directory");
        qmlRegisterType<File>(uri, 1, 0, "File");
    }

関数 ``registerTypes()`` は私達のクラス ``File`` と ``Directory`` をQMLに登録します。この関数は、テンプレートのクラス名、メジャーバージョン番号、マイナーバージョン番号、およびクラス名を必要とします。コメント ``@uri <module identifier>`` により、Qt CreatorにこのモジュールをインポートしているQMLファイルを編集している時、登録した型を知らせる事ができます。


C++クラスにQMLプロパティを作成
------------------------------

C++と `QtのMeta-Objectシステム`_ を使って、QML型とプロパティを作ることが出来ます。プロパティを実装するために、Qtにそれらのプロパティを認識させる、スロット・アンド・シグナルを使います。それらのプロパティはQMLで使うことが出来るようになります。

テキストエディタのためには、ファイルの読込と保存が出来る必要があります。通常、それらの機能はファイルダイアログに含まれています。幸運なことに、 `QDir`_ 、 `QFile`_ 、および `QTextStream`_ が、ディレクトリーの読み込みや、ストリーム入力・出力の実装に使えます。

::

    class Directory : public QObject {
        Q_OBJECT

        Q_PROPERTY (int filesCount READ filesCount CONSTANT)
        Q_PROPERTY (QString filename READ filename WRITE setFilename NOTIFY filenameChanged)
        Q_PROPERTY (QString fileContent READ fileContent WRITE setFileContent NOTIFY fileContentChanged)
        Q_PROPERTY (QQmlListProperty<File> files READ files CONSTANT)
        ...

クラス ``Directory`` は、 QtのMeta-Objectシステム をファイルハンドリングを必要とするプロパティを登録するために使っています。クラス ``Directory`` はプラグインとしてエクスポートされ、QMLで ``Directory``
型として使用可能です。マクロ ``Q_PROPERTY`` を使ってリストした各プロパティは、QMLプロパティです。

`Q_PROPERTY`_ は QtのMeta-Objectシステム へ渡す読込・書込関数を宣言することで、プロパティを宣言します。例えば、プロパティ ``filename`` は、型は `QString`_ で、読込は関数 ``filename()`` を使用可能で、書込は関数 ``setFilename()`` を使用可能です。加えて、プロパティ ``filename`` と関連付けられたシグナル、 ``filenameChanged()`` が在り、そのプロパティが変更されるたびに発行されます。読込・書込関数は ``public`` としてヘッダーファイルで宣言されています。

同様に、私達は他のプロパティも用途に応じて宣言しています。プロパティ ``filesCount`` はディレクトリ内のファイルの数を示し、プロパティ ``filename`` は現在選択中のファイルの名前、プロパティ ``fileContent`` は読み込んだ・書き込んだファイルの中身を持ちます。

::

    Q_PROPERTY(QQmlListProperty<File> files READ files CONSTANT)

リストプロパティ ``files`` は、ディレクトリ内のすべてのフィルタされたファイルのリストです。クラス ``Directory`` は無効なテキストファイルを除外するように実装されており、 ``.txt`` 拡張子のファイルだけが有効です。さらに、 `QList`_ はC++で `QQmlListProperty`_ として宣言することで、QMLファイルの中で使えます。そのテンプレート引数として取られるクラスは `QObject`_ から継承したものである必要があり、したがってクラス ``File`` も `QObject`_ から継承しなければなりません。クラス ``Directory`` では、 ``File`` オブジェクトのリストが ``m_fileList`` と名づけた `QList`_ に格納されています。

::

    class File : public QObject{

        Q_OBJECT
        Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)

        ...
    };

これで、QMLから ``Directory`` オブジェクトのプロパティの一部としてそれらのプロパティを使えるようになります。

.. note::
   C++コードで識別子 ``id`` を作成する必要はありません。

::

    Directory {
        id: directory

        filesCount
        filename
        fileContent
        files

        files[0].name
    }

QMLはJavascriptの構文と構造を使っているので、ファイルのリストを反復処理し、そのプロパティを取得することが出来ます。最初のファイルのプロパティ ``name`` を取得するために、 ``files[0].name`` を呼ぶ事が出来ます。

通常のC++関数も、QMLよりアクセス可能です。ファイル読込・書込関数はC++で実装され、 `Q_INVOKABLE`_ マクロを使って宣言されています。私達は代わりに、 ``slot`` とQMLよりアクセス可能な関数、として関数を宣言できます。

directory.h より::

    Q_INVOKABLE void saveFile();
    Q_INVOKABLE void loadFile();

クラス ``Directory`` も、ディレクトリの内容が変更されるたびに他のオブジェクトに通知しなければなりません。この機能は ``signal`` を用いて行われます。前述のように、QMLシグナルは、対応するその名前の前に ``on`` を付けた名前のハンドラーを持ちます。 ``directoryChanged`` と名付けられたシグナルは、ディレクトリの更新があるたびに呼び出されます。更新は単にディレクトリの内容を再読み込みし、ディレクトリの有効なファイルの一覧を更新します。シグナルハンドラー ``onDirectoryChanged`` へアクションをアタッチすることで、QML項目は更新を通知されます。

リストプロパティは更に検討する必要があります。これはリストプロパティがコールバックをリストの内容にアクセスおよび変更するために使うからです。このリストプロパティは ``QQmlListProperty<File>`` 型です。そのリストがアクセスされるたび、そのアクセサ関数は ``QQmlListProperty<File>`` を返す必要があります。テンプレート型 ``File`` は ``QObject`` の派生である必要があります。更に、 ``QQmlListProperty`` を作るには、リストのアクセサとモディファイアがコンストラクタに関数ポインターとして渡されている必要があります。そのリスト、私達の場合は ``QList`` も、 ``File`` へのポインターのリストである必要があります。

`QQmlListProperty`_ のコンストラクタは、次のように宣言されています::

    QQmlListProperty (QObject *object, void *data, AppendFunction append,
                      CountFunction count = 0, AtFunction at = 0, ClearFunction clear = 0);

リストへ追加、リストをカウント、添字によって要素を取得、およびリストを空にする関数へのポインターをとっています。関数 ``append`` だけが必須です。

.. note::
   関数ポインターはそれぞれ `AppendFunction`_ 、 `CountFunction`_ 、 `AtFunction`_ 、 `ClearFunction`_ の定義と一致していなければなりません。

クラス ``Directory`` は `QQmlListProperty`_ インスタンスをこのように作ります::

    QQmlListProperty<File>(this, &m_fileList, &appendFiles, &filesSize, &fileAt, &clearFilesPtr);

引数のポインターは次の関数を指しています::

    void appendFiles(QQmlListProperty<File> *property, File *file);
    File* fileAt(QQmlListProperty<File> *property, int index);
    int filesSize(QQmlListProperty<File> *property);
    void clearFilesPtr(QQmlListProperty<File> *property);

私達のファイルダイアログを簡単にするため、クラス ``Directory`` は ``.txt`` 拡張子を持たない無効なテキストファイルを除外します。もしファイル名が ``.txt`` 拡張子を持たないのであれば、私達のファイルダイアログにそれは映りません。また、その実装では保存したファイルが ``.txt`` 拡張子をファイル名に持つか確かめます。 ``Directory`` は ``QTextStream`` をファイルの読込およびファイルの内容のファイルへの出力のために使います。

私達の ``Directory`` オブジェクトで、ファイルたちをリストとして取得でき、いくつのテキストファイルがアプリケーションディレクトリにあるか知ることができ、ファイルの名前と内容を文字列として取得でき、ディレクトリの内容に変更が有るたびに通知されることができます。

プラグインをビルドするには、 ``qmake`` を ``filedialog.pro`` で実行し、そして ``make`` を実行してビルドと ``plugins`` ディレクトリへのプラグインの転送を行います。

.. _`QtのMeta-Objectシステム`: http://qt-project.org/doc/qt-5/metaobjects.html
.. _`QDir`: http://qt-project.org/doc/qt-5/qdir.html
.. _`QFile`: http://qt-project.org/doc/qt-5/qfile.html
.. _`QTextStream`: http://qt-project.org/doc/qt-5/qtextstream.html
.. _`Q_PROPERTY`: http://qt-project.org/doc/qt-5/qobject.html#Q_PROPERTY
.. _`QString`: http://qt-project.org/doc/qt-5/qstring.html
.. _`QList`: http://qt-project.org/doc/qt-5/qlist.html
.. _`QQmlListProperty`: http://qt-project.org/doc/qt-5/qqmllistproperty.html
.. _`QObject`: http://qt-project.org/doc/qt-5/qobject.html
.. _`Q_INVOKABLE`: http://qt-project.org/doc/qt-5/qobject.html#Q_INVOKABLE
.. _`AppendFunction`: http://qt-project.org/doc/qt-5/qqmllistproperty.html#AppendFunction-typedef
.. _`CountFunction`: http://qt-project.org/doc/qt-5/qqmllistproperty.html#CountFunction-typedef
.. _`AtFunction`: http://qt-project.org/doc/qt-5/qqmllistproperty.html#AtFunction-typedef
.. _`ClearFunction`: http://qt-project.org/doc/qt-5/qqmllistproperty.html#ClearFunction-typedef


QMLでのプラグインのインポート
---------------------------

ツール ``qmlscene`` は同じディレクトリにあるファイルをアプリケーションとしてインポートします。インポートしたい内容の位置を含むファイル ``qmldir`` を作ります。今回の場合、プラグインだけあるのですが、他のリソース（QML型、JavaScriptファイル）も ``qmldir`` で同様にうまく定義されることが出来ます。

ファイル qmldir の内容::

    module FileDialog
    plugin filedialogplugin

先ほど作成したモジュールは ``FileDialog`` と呼ばれ、プロジェクトファイルの ``TARGET`` フィールドと同じ ``filedialogplugin`` と呼ばれるプラグインを利用可能にします。プラグインへのパスを定義しなかったので、QMLエンジンはファイル ``qmldir`` と同じディレクトリからそれを見つけると期待します。

私達により登録されたQML型を、QMLからインポートすることが出来るようになりました::

    import FileDialog 1.0

    Directory {
        id: directory
    }
    ...


ファイルダイアログのファイルメニューへの統合
--------------------------------------------

私達の ``FileMenu`` は、ディレクトリ内のテキストファイルのリストを含む ``FileDialog`` オブジェクトを表示して、ユーザーがリストをクリックすることでファイルを選べるようにする必要があります。また、読込、書込、新規作成ボタンに、それぞれに期待される動作を割り当てる必要があります。 ``FileMenu`` は編集可能なユーザーがキーボードでファイル名をタイプ出来るように、テキスト入力を含みます。

``Directory`` オブジェクトはファイル ``FileMenu.qml`` で使われ、 ``FileDialog`` オブジェクトにディレクトリの内容が更新されたことを通知します。この通知はシグナルハンドラーである ``onDirectoryChanged`` で行われます。

FileMenu.qml より::

    Directory {
        id: directory
        filename: textInput.text
        onDirectoryChanged: fileDialog.notifyRefresh()
    }

私達のアプリケーションの簡単さを保つため、ファイルダイアログは常に可視で、 ``.txt`` 拡張子をファイル名に持たない無効なテキストファイルは表示しません。

FileDialog.qml より::

    signal notifyRefresh()
    onNotifyRefresh: dirView.model = directory.files

``FileDialog`` オブジェクトは、そのリストプロパティ ``files`` を読むことでディレクトリの内容を表示します。 ``files`` は、デリゲートによりデータの項目をグリッドに表示する `GridView`_ オブジェクトのモデルとして使われます。デリゲートはモデルの外観をハンドルし、私達のファイルダイアログは単純に中央に置かれたテキストのグリッドです。ファイル名をクリックするとその結果、矩形の外観がファイル名がハイライトされたものになります。 ``FileDialog`` はシグナル ``notifyRefresh`` が発行されるたびに通知され、ディレクトリ内のファイルたちを再読み込みします。

FileMenu.qml より::

    Button {
        id: newButton
        label: "New"
        onButtonClick: {
            textArea.textContent = ""
        }
    }
    Button {
        id: loadButton
        label: "Load"
        onButtonClick: {
            directory.filename = textInput.text
            directory.loadFile()
            textArea.textContent = directory.fileContent
        }
    }
    Button {
        id: saveButton
        label: "Save"
        onButtonClick: {
            directory.fileContent = textArea.textContent
            directory.filename = textInput.text
            directory.saveFile()
        }
    }
    Button {
        id: exitButton
        label: "Exit"
        onButtonClick: {
            Qt.quit()
        }
    }

``FileMenu`` は今やそれぞれの期待される動作と接続されています。 ``saveButton`` はテキストを ``TextEdit`` から ``directory`` のプロパティ ``fileContent`` へと渡し、続いて編集可能なテキスト入力からそれのファイル名をコピーします。最後に、そのボタンは関数 ``saveFile()`` を呼び、ファイルを保存します。 ``loadButoon`` は同様の実行を持ちます。また、 ``New`` の動作は ``TextEdit`` の内容を空にします。

さらに、 ``EditMenu`` のボタンはコピー、貼り付け、全選択といった ``TextEdit`` の関数とそれぞれ接続されています。

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor5_filemenu.png

.. `GridView`_: http://qt-project.org/doc/qt-5/qml-qtquick-gridview.html


最終的なテキストエディタアプリケーション
========================================

.. image:: http://qt-project.org/doc/qt-5/images/qml-texteditor5_newfile.png

アプリケーションは簡単なテキストエディタとして機能でき、テキストを受け、ファイルへ保存することが出来ます。また、ファイルを読み込み、テキスト操作を行うこともできます。


テキストエディタの実行
======================

テキストエディタを実行する前に、ファイルダイアログのC++プラグインをビルドする必要があります。それをビルドしたら、ディレクトリ ``filedialog`` に入って、 ``qmake`` を実行し、 ``make`` または ``nmake`` をプラットフォームに合わせて用いてコンパイルしてください。

テキストエディタを `qmlscene`_ を、インポートディレクトリを引数として渡してQMLエンジンに私達のファイルダイアログプラグインのモジュールをどこから探せばいいか分からせて、実行します。::

    qmlscene -I ./imports texteditor.qml

完全なソースコードはディレクトリ ``examples/quick/tutorials/gettingStartedQml`` にあります。
