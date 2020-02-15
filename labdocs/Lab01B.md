# Lab 1B: Azure Machine Learning Toolsの使用

このラボでは、Azure Machine Learningワークスペースを操作するためのさまざまなツールを探索します。

## 始める前に

このラボを開始する前に、[前のラボ]（Lab01A.md）の指示に従ってAzure Machine Learningワークスペースを作成しておく必要があります。

## Task 1: コンピューティングインスタンスでAzure ML SDKを使用する

ほとんどの資産管理タスクを実行して、* Studio *インターフェースで環境をセットアップできますが、構成タスクをスクリプト化して繰り返しや自動化を容易にすることも重要です。

1. [Azure Machine Learning studio](https://ml.azure.com)のワークスペースの[**Compute**]ページで、[**Compute Instances**]タブを表示し、必要に応じて[**Refresh**]をクリックします前のラボで作成したコンピューティングインスタンスが起動するまで定期的に。次に、**Jupyter**リンクをクリックして、VMでJupyterノートブックを開きます。

2. ノートブック環境で、新しい**ターミナル**を作成します。これにより、コマンドシェルで新しいタブが開きます。

3. Azure Machine Learning SDKは既にコンピューティングインスタンスイメージにインストールされていますが、このコースで必要なオプションパッケージとともに最新バージョンを使用することをお勧めします。次のコマンドを入力して、SDKパッケージを更新します:

    ```bash
    pip install --upgrade azureml-sdk[notebooks,automl,explain]
    ```

    > **詳しくは**: Azure ML SDKとそのオプションコンポーネントのインストールの詳細については、[Azure ML SDKドキュメント](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py)を参照してください。 

4. 次に、次のコマンドを実行して現在のディレクトリを**Users**ディレクトリに変更し、このコースのラボで使用するノートブックを取得します:

    ```bash
    cd Users
    git clone https://github.com/tottokug/DP100
    ```

5. コマンドが完了したら、ターミナルタブを閉じて、Jupyterノートブックファイルエクスプローラーでホームページを表示します。次に、**Users**フォルダーを開きます。このフォルダーには**DP100**フォルダーが含まれている必要があり、このラボの残りの部分で使用するファイルが含まれています。
6. **Users/DP100**フォルダーで、**01B-Azure ML SDK.ipynb**ノートブックの紹介を開きます。次に、ノートブックのメモを読み、各コードセルを順番に実行します。

## Task 2: Visual Studioオンライン環境をセットアップする

Azure Machine Learningのコンピューティングインスタンスは、独自のPythonインストールを管理する必要なく、Azure MLで作業するための管理しやすいPython環境を提供します。ただし、独自のグラフィカルPython開発環境を使用する場合があります。このコースでは、インストールを簡素化するためにVisual Studio Onlineを使用しますが、Azure Machine Learning SDKを使用する原則は、どのPython環境でも同じです。

> **Note**: 執筆時点では、Visual Studio Onlineは*プレビュー*状態です。予期しないエラーメッセージが表示される場合があります。

1. 新しいブラウザタブで、[https://online.visualstudio.com](https://online.visualstudio.com)に移動し、**Get Started**をクリックします。
2. Azureへのサインインに使用したのと同じMicrosoft資格情報を使用してVisual Studio Onlineにサインインします。
3. 次の設定で新しい環境を作成し、プロンプトが表示されたら最初にAzureサブスクリプションで請求プランを作成します:
    - **Environment Name**: *お好みの一意の名前*
    - **Git Repository**: tottokug/DP100
    - **Instance Type**: Standard (Linux)
    - **suspend idle environment after**: 30 Minutes
4. 環境が作成されるのを待ってから、その名前をクリックして接続します。

    Visual Studio Onlineは、Webブラウザーで使用できるVisual Studio Codeのホストされたインスタンスです。 Visual Studio Codeは一般的なコード編集環境であり、*拡張機能*のインストールによりさまざまなプログラミング言語をサポートしています。 Pythonを使用するには、**DP100**リポジトリからこの環境を作成したときに一般的に使用されるいくつかのPythonパッケージとともにインストールされたMicrosoft Python拡張が必要です。

    ホストされているVisual Studio Code環境には、Pythonの3つのインストール（バージョン2.7.13、3.5.3、および3.8.0）が含まれています。 Python **3.5.3**仮想環境を使用します。独自のインストールでは、Pythonのインストール、仮想環境の作成、必要なパッケージのインストールを担当します。この実習ラボでは、一般的なPython構成のほとんどが完了していますが、Azure Machine Learning SDKをインストールする必要があります。

5. Visual Studio Online環境では、DP100リポジトリのコンテンツがロードされるのを待ってから、アプリケーションメニュー（**☰**）の**表示**メニューで、**コマンドパレット**（または、Ctrl + Shift + Pキーを押します。次に、パレットでコマンド** Python：Create Terminal **を入力します。これにより、Visual Studio Onlineインターフェイスの下部にPythonターミナルペインが開きます。プロンプトが表示されたら、** Python 3.5.3 **インタープリターを選択します。

6. ターミナルペインで、次のコマンドを入力して、Python 3.5.3仮想環境が定義されているディレクトリに移動します。:

    ````bash
    cd /usr/bin
    ````

7. このコマンドを使用して、Azure Machine Learning SDKをインストールします（オプションの*notebooks*追加パッケージを使用）:

    ```bash
    sudo pip install azureml-sdk[notebooks]
    ```

8. ターミナルペインを閉じます。

## Task 3: Visual Studio OnlineでAzure ML SDKを使用する

Python開発環境ができたので、その中でAzure Machine Learning SDKを使用できます。最初に、Azure Machine Learningワークスペースに接続するために必要な構成情報を取得する必要があります。

1. 新しいブラウザータブで、[https://portal.azure.com](https://portal.azure.com)でAzureポータルを開き、必要に応じてサインインします。
2. 前のラボで作成したAzure Machine Learningワークスペースリソースを開き、**概要**ページで**Download config.json**をクリックして、ファイルをローカルコンピューターにダウンロードします。
3. ダウンロードした** config.json **ファイルをテキストエディターで開き、その内容をクリップボードにコピーします。このファイルには、ワークスペースに接続するために必要な構成情報が含まれています。
4. Visual Studio Onlineで、VS Onlineワークスペースのルートフォルダーに**config.json**という名前の新しいファイルを作成します。
5. コピーした構成情報をVisual Studio Onlineワークスペースの新しいconfig.jsonファイルに貼り付けて保存します。
6. Visual Studio Onlineで、** 01B-Azure ML SDK.ipynb **ノートブックの概要を開きます。これは、Visual Studio Online内のJupyter Notebookインターフェイスに読み込まれます。 Jupyter Notebooksインターフェースを初めて使用する場合、ロードに時間がかかる場合があります。2つのペインが簡単に表示される場合があります。
7. ノートブックがロードされたら、Visual Studio Onlineインターフェースの左下で、現在のPython仮想環境をクリックします。これは、リポジトリの構成設定に基づいて**Python 3.5.3**に変更されているはずですが、とにかくその仮想環境を再度選択します（ノートブックは、メタデータに示されている別のバージョンで作成されています）。
8. Azure Machine Learning Notebook VM Jupyter環境で行ったように、ノートブックのノートを読み、各コードセルを順番に実行します。

## Task 4: Visual Studio Code Azure Machine Learning Extensionを使用する

Visual Studio Online（またはVisual Studio Codeのローカルインストール）でAzure Machine Learningを使用する予定の場合、Azure Machine Learning拡張機能を使用すると、コード開発環境を切り替えることなくワークスペースのリソースを簡単に操作できますAzure Machine Learning Studio Webインターフェイス。

1. Visual Studio Onlineで[**拡張機能**]タブ（⊞）をクリックし、「Azure Machine Learning」を検索します。次に、Microsoftの**Azure Machine Learning**拡張機能をインストールします。拡張機能がインストールされたら、**Reload Required**ボタンをクリックして、拡張機能を使用して環境をリロードします。
2. Visual Studio Onlineで**Azure**タブ（**&Delta;**）をクリックし、**Azure Machine Learning**セクションでサブスクリプションとAzure Machine Learningワークスペースを展開します。
3. ** Compute **を展開し、ワークスペースで作成した** aml-cluster **コンピューティングリソースが、** local **コンピューティングリソースと共にリストされていることを確認します。この場合、Visual Studio Onlineホスト環境を表します。ローカルコンピューティングおよびワークスペースで定義されたコンピューティングリソースでAzure Machine Learningコード実験を実行できます。
4. Visual Studio Onlineブラウザータブを閉じます。

> **Note**: [次の演習]（Lab02A.md）に直接進む場合は、コンピューティングインスタンスを実行したままにします。休憩する場合は、すべてのJupyterタブを閉じてコンピューティングインスタンスを**停止**し、不要なコストが発生しないようにすることができます。