.. -*- coding: utf-8 -*-

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

.. code-block:: qml

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

.. code-block:: qml

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

.. code-block:: qml

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
