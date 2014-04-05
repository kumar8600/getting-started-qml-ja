.. -*- coding: utf-8 -*-

テキストエディタの装飾
======================


引き出しインターフェースの実装
------------------------------

私達のテキストエディタはシンプルに見えますし、それを装飾する必要があります。QMLを使って、私達のテキストエディタの遷移を宣言したり、アニメーションさせたり出来ます。私達のメニューバーは画面の３分の１を占領しているので、欲しい時だけ姿を見せてくれると良いでしょう。

メニューバーがクリックされたとき伸び縮みする、引き出しインターフェースを追加できます。私達の実装では、マウスクリックに反応する細い矩形を持ちます。 ``drawer`` は、引き出しと同じように、２つの状態を持ちます。状態「引き出しは開いている」と、状態「引き出しは閉じている」です。項目 ``drawer`` は、高さが小さく細長い矩形です。入れ子になった、矢印アイコンを表す `Image`_ オブジェクトが、引き出しの内側の中央に配置されるよう宣言されています。引き出しは、ユーザーにマウスエリアをクリックされるたびに、識別子 ``screen`` でアプリケーションの全体へ、状態を代入します。

.. _`Image`: http://qt-project.org/doc/qt-5/qml-qtquick-image.html

.. code-block:: qml

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

.. code-block:: qml

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

.. code-block:: qml

    transitions: [
        Transition {
            to: "*"
            NumberAnimation { target: textArea; properties: "y, height"; duration: 100; easing.type:Easing.OutExpo }
            NumberAnimation { target: menuBar; properties: "y"; duration: 100; easing.type: Easing.OutExpo }
            NumberAnimation { target: drawer; properties: "y"; duration: 100; easing.type: Easing.OutExpo }
        }
    ]

プロパティの変化をアニメーションさせるもう一つの方法は、 `Behavior`_ 型を宣言することです。遷移は状態変化時にのみ動作し、そして ``Behavior`` は一般的なプロパティ変化のアニメーションを設定できます。テキストエディタでは、矢印が ``NumberAnimation`` を持ち、プロパティ ``rotation`` の変化をアニメーションさせます。

.. _`Behavior`: http://qt-project.org/doc/qt-5/qml-qtquick-behavior.html

TextEditor.qml より:

.. code-block:: qml

    Behavior {
        NumberAnimation { property: "rotation"; easing.type: Easing.OutExpo }
    }

私達の部品と、状態とアニメーションの知識の話に戻りましょう。私達は部品の外観をより良くできます。 ``Button.qml`` では、 ``color`` を追加し、プロパティ ``scale`` をボタンがクリックされた時に変化させることが出来ます。色型は `ColorAnimation`_ を用いてアニメーションさせることが出来、数は `NumberAnimation`_ で出来ます。以下で示している構文 ``on propertyName`` は、唯一のプロパティをターゲットとする時役に立ちます。

.. _`ColorAnimation`: http://qt-project.org/doc/qt-5/qml-qtquick-coloranimation.html
.. _`NumberAnimation`: http://qt-project.org/doc/qt-5/qml-qtquick-numberanimation.html

Button.qml より:

.. code-block:: qml

    ...

    color: buttonMouseArea.pressed ? Qt.darker(buttonColor, 1.5) : buttonColor
    Behavior on color { ColorAnimation{ duration: 55 } }

    scale: buttonMouseArea.pressed ? 1.1 : 1.0
    Behavior on scale { NumberAnimation{ duration: 55 } }

加えて、QMLの部品の外観を向上させるために、グラデーションのようなカラーエフェクトや透明度エフェクトが使えます。 `Gradient`_ オブジェクトを宣言すると、プロパティ ``color`` は上書きされます。グラデーションの色は、 `GradientStop`_ 型を使って宣言できます。グラデーションは ``0.0`` から ``1.0`` までの間の比率で位置づけられます。

.. _`Gradient`: http://qt-project.org/doc/qt-5/qml-qtquick-gradient.html
.. _`GradientStop`: http://qt-project.org/doc/qt-5/qml-qtquick-gradientstop.html

MenuBar.qml より:

.. code-block:: qml

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

