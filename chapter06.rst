.. -*- coding: utf-8 -*-

Qt C++を用いたQMLの拡張
=======================

テキストエディタのレイアウトが出来たので、今C++でテキストエディタの機能を実装することが出来ます。QMLとC++を共に使うことで、Qtで私達のアプリケーションロジックを作ることが可能となります。C++アプリケーションで `QtのQuickクラス`_ を用いることで、QMLコンテクストを作成でき、そして `QQuickView`_ を用いてQML型を表示する事ができます。代わりに、C++コードを拡張プラグインへとエクスポートして、新たな `identified module`_ としてQMLからアクセス出来るようにすることも出来ます。 `qmlscene`_ でQMLファイルを起動するとき、 `import paths`_ の一つからモジュールが見つかることさえ保証されていれば良いです。私達のアプリケーションでは、後者のアプローチを採ります。こうして、実行可能ファイルを実行するのではなく、QMLファイルを ``qmlscene`` から直接読み込むことが出来るのです。

.. _`QtのQuickクラス`: http://qt-project.org/doc/qt-5/qtqml-cppintegration-topic.html
.. _`QQuickView`: http://qt-project.org/doc/qt-5/qquickview.html
.. _`identified module`: http://qt-project.org/doc/qt-5/qtqml-modules-identifiedmodules.html
.. _`qmlscene`: http://qt-project.org/doc/qt-5/qtquick-qmlscene.html
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
   
   .. _`Qt Quick Dialogs`: http://qt-project.org/doc/qt-5/qtquickdialogs-index.html


Qtプラグインのビルド
--------------------

プラグインをビルドするには、Qtプロジェクトファイルに次のように設定する必要があります。まず必要なソース、ヘッダー、およびQtモジュールを私達のプロジェクトファイルに追加する必要があります。すべてのC++コードとプロジェクトファイルは ``filedialog`` ディレクトリにあります。

filedialog.pro より:

.. code::

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

dialogPlugin.h より:

.. code-block:: c++

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
.. _`registerTypes()`: http://qt-project.org/doc/qt-5/qqmlextensionplugin.html#registerTypes

DialogPlugin.cpp より:

.. code-block:: c++

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

.. code-block:: c++

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

.. code-block:: c++

    Q_PROPERTY(QQmlListProperty<File> files READ files CONSTANT)

リストプロパティ ``files`` は、ディレクトリ内のすべてのフィルタされたファイルのリストです。クラス ``Directory`` は無効なテキストファイルを除外するように実装されており、 ``.txt`` 拡張子のファイルだけが有効です。さらに、 `QList`_ はC++で `QQmlListProperty`_ として宣言することで、QMLファイルの中で使えます。そのテンプレート引数として取られるクラスは `QObject`_ から継承したものである必要があり、したがってクラス ``File`` も `QObject`_ から継承しなければなりません。クラス ``Directory`` では、 ``File`` オブジェクトのリストが ``m_fileList`` と名づけた `QList`_ に格納されています。

.. code-block:: c++

    class File : public QObject{

        Q_OBJECT
        Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)

        ...
    };

これで、QMLから ``Directory`` オブジェクトのプロパティの一部としてそれらのプロパティを使えるようになります。

.. note::
   C++コードで識別子 ``id`` を作成する必要はありません。

.. code-block:: c++

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

directory.h より:

.. code-block:: c++

    Q_INVOKABLE void saveFile();
    Q_INVOKABLE void loadFile();

クラス ``Directory`` も、ディレクトリの内容が変更されるたびに他のオブジェクトに通知しなければなりません。この機能は ``signal`` を用いて行われます。前述のように、QMLシグナルは、対応するその名前の前に ``on`` を付けた名前のハンドラーを持ちます。 ``directoryChanged`` と名付けられたシグナルは、ディレクトリの更新があるたびに呼び出されます。更新は単にディレクトリの内容を再読み込みし、ディレクトリの有効なファイルの一覧を更新します。シグナルハンドラー ``onDirectoryChanged`` へアクションをアタッチすることで、QML項目は更新を通知されます。

リストプロパティは更に検討する必要があります。これはリストプロパティがコールバックをリストの内容にアクセスおよび変更するために使うからです。このリストプロパティは ``QQmlListProperty<File>`` 型です。そのリストがアクセスされるたび、そのアクセサ関数は ``QQmlListProperty<File>`` を返す必要があります。テンプレート型 ``File`` は ``QObject`` の派生である必要があります。更に、 ``QQmlListProperty`` を作るには、リストのアクセサとモディファイアがコンストラクタに関数ポインターとして渡されている必要があります。そのリスト、私達の場合は ``QList`` も、 ``File`` へのポインターのリストである必要があります。

`QQmlListProperty`_ のコンストラクタは、次のように宣言されています:

.. code-block:: c++

    QQmlListProperty (QObject *object, void *data, AppendFunction append,
                      CountFunction count = 0, AtFunction at = 0, ClearFunction clear = 0);

リストへ追加、リストをカウント、添字によって要素を取得、およびリストを空にする関数へのポインターをとっています。関数 ``append`` だけが必須です。

.. note::
   関数ポインターはそれぞれ `AppendFunction`_ 、 `CountFunction`_ 、 `AtFunction`_ 、 `ClearFunction`_ の定義と一致していなければなりません。

クラス ``Directory`` は `QQmlListProperty`_ インスタンスをこのように作ります:

.. code-block:: c++

    QQmlListProperty<File>(this, &m_fileList, &appendFiles, &filesSize, &fileAt, &clearFilesPtr);

引数のポインターは次の関数を指しています:

.. code-block:: c++

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
-----------------------------

ツール ``qmlscene`` は同じディレクトリにあるファイルをアプリケーションとしてインポートします。インポートしたい内容の位置を含むファイル ``qmldir`` を作ります。今回の場合、プラグインだけあるのですが、他のリソース（QML型、JavaScriptファイル）も ``qmldir`` で同様にうまく定義されることが出来ます。

ファイル qmldir の内容:

.. code-block:: c++

    module FileDialog
    plugin filedialogplugin

先ほど作成したモジュールは ``FileDialog`` と呼ばれ、プロジェクトファイルの ``TARGET`` フィールドと同じ ``filedialogplugin`` と呼ばれるプラグインを利用可能にします。プラグインへのパスを定義しなかったので、QMLエンジンはファイル ``qmldir`` と同じディレクトリからそれを見つけると期待します。

私達により登録されたQML型を、QMLからインポートすることが出来るようになりました:

.. code-block:: c++

    import FileDialog 1.0

    Directory {
        id: directory
    }
    ...


ファイルダイアログのファイルメニューへの統合
--------------------------------------------

私達の ``FileMenu`` は、ディレクトリ内のテキストファイルのリストを含む ``FileDialog`` オブジェクトを表示して、ユーザーがリストをクリックすることでファイルを選べるようにする必要があります。また、読込、書込、新規作成ボタンに、それぞれに期待される動作を割り当てる必要があります。 ``FileMenu`` は編集可能なユーザーがキーボードでファイル名をタイプ出来るように、テキスト入力を含みます。

``Directory`` オブジェクトはファイル ``FileMenu.qml`` で使われ、 ``FileDialog`` オブジェクトにディレクトリの内容が更新されたことを通知します。この通知はシグナルハンドラーである ``onDirectoryChanged`` で行われます。

FileMenu.qml より:

.. code-block:: c++

    Directory {
        id: directory
        filename: textInput.text
        onDirectoryChanged: fileDialog.notifyRefresh()
    }

私達のアプリケーションの簡単さを保つため、ファイルダイアログは常に可視で、 ``.txt`` 拡張子をファイル名に持たない無効なテキストファイルは表示しません。

FileDialog.qml より:

.. code-block:: c++

    signal notifyRefresh()
    onNotifyRefresh: dirView.model = directory.files

``FileDialog`` オブジェクトは、そのリストプロパティ ``files`` を読むことでディレクトリの内容を表示します。 ``files`` は、デリゲートによりデータの項目をグリッドに表示する `GridView`_ オブジェクトのモデルとして使われます。デリゲートはモデルの外観をハンドルし、私達のファイルダイアログは単純に中央に置かれたテキストのグリッドです。ファイル名をクリックするとその結果、矩形の外観がファイル名がハイライトされたものになります。 ``FileDialog`` はシグナル ``notifyRefresh`` が発行されるたびに通知され、ディレクトリ内のファイルたちを再読み込みします。

FileMenu.qml より:

.. code-block:: c++

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

.. image:: ./images/qml-texteditor5_filemenu.png

.. _`GridView`: http://qt-project.org/doc/qt-5/qml-qtquick-gridview.html

