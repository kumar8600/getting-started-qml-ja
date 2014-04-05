.. -*- coding: utf-8 -*-

テキストエディタの構築
======================


TextAreaの宣言
--------------

編集できるテキストエリアのないテキストエディタなんて、テキストエディタとは呼べません。QMLの `TextEdit`_ 型は、複数行の編集できるテキストエリアを可能にします。 ``TextEdit`` は、直接ユーザーにテキストを編集することを許さない `Text`_ 型とは異なります。

.. _`Text`: http://qt-project.org/doc/qt-5/qml-qtquick-text.html
.. _`TextEdit`: http://qt-project.org/doc/qt-5/qml-qtquick-textedit.html

.. code-block:: qml

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

.. code-block:: javascript

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

.. code-block:: qml

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

