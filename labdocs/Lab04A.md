# Lab 4A: データストアの使用

データサイエンティストがローカルファイルシステムのデータを操作することはかなり一般的ですが、エンタープライズ環境では、複数のデータサイエンティストがアクセスできる中央の場所にデータを保存する方が効果的です。このラボでは、データをクラウドに保存し、Azure Machine Learning * datastore *を使用してアクセスします。

## 始める前に

このラボを開始する前に、[ラボ1A](Lab01A.md)および[ラボ1B](Lab01B.md)を完了していることを確認してください。これらには、このラボで使用するAzure Machine Learningワークスペースおよびその他のリソースを作成するタスクが含まれています。

## Task 1: Azureストレージコンテナーを作成する
Azure Machine Learningでは、さまざまな種類のAzureデータソースを使用できます。このラボでは、blobコンテナーを含むAzureストレージアカウントを作成します。

1. [Azureポータル](https://portal.azure.com)にサインインし、Azure Machine Learningワークスペースを含むリソースグループを開きます。これには、ワークスペースで作成されたAzureストレージアカウントが既に含まれていることに注意してください。

    >**Note**: ワークスペースで作成されたストレージアカウントは、構成データ、ノートブック、登録済みモデルなどを保存するためにサービスによって使用されます。また、実験およびモデルトレーニング用のデータを保存するために使用できますが、多くの場合、このデータを個別に管理する必要があります。

2. 次の設定で、リソースグループに新しい**Storage Account**を追加します:

    - **Storage account name**: 一意の名前。
    - **Location**: ワークスペースと同じ場所
    - **Performance**: Standard
    - **Account kind**: StorageV2 (general purpose v2)
    - **Access tier (default)**: Hot
    - ネットワークのデフォルト設定を使用する

3. ストレージアカウントが作成されるのを待ってから、ポータルのリソースに移動します。
4. ストレージアカウントの**Containers**ページを開き、次の設定でコンテナーを追加します。

    - **Name**: aml-data
    - **Public access level**: Private (no anonymous access)

5. コンテナを追加したら、ストレージアカウントの**Access Key**ページを表示し、**key1**をクリップボードにコピーします。これは次のタスクで必要になります。

## Task 2: Azure Machine Learning Datastoreを登録する

ストレージコンテナーを作成したら、Azure Machine Learningワークスペースでデータストアとして登録できます。

1. [Azure Machine Learning studio](https://ml.azure.com)で、ワークスペースの**Datastores**ページを表示します。事前定義されたデータストアがリストされます。
2. 次の設定で新しいデータストアを作成します:
    - **Datastore name**: aml_data
    - **Datastore type**: Azure Blob Storage
    - **Account selection method**: From Azure subscription
    - **Subscription ID**: *Your Azure subscription*
    - **Storage account**: *前のタスクで作成したストレージアカウント*
    - **Blob container**: aml-data
    - **Authentication type**: Account key
    - **Account key**: *前のタスクでコピーしたキーを貼り付けます*
3. データストアを追加したら、[**Datastore**]ページにリストされていることを確認します。

## Task 3: Azure Machine Learning SDKを使用してデータストアにアクセスする

このタスクでは、Azure Machine Learning SDKを使用して、データストアとの間でデータをアップロードおよびダウンロードします。

1. [Azure Machine Learning studio](https://ml.azure.com)で、ワークスペースの**Compute**ページを表示します。 [** Compute Instances **]タブで、コンピューティングインスタンスが実行されていることを確認します。そうでない場合は、開始します。
2. コンピューティングインスタンスの実行中に、**Jupyter**リンクをクリックして、新しいブラウザータブでJupyterホームページを開きます。
3. Jupyterホームページの**Users/DP100**フォルダーで、**04A-Working with Datastores.ipynb**ノートブックを開きます。次に、ノートブックのメモを読み、各コードセルを順番に実行します。

> **Note**: [次の演習](Lab04B.md)に直接進む場合は、コンピューティングインスタンスを実行したままにします。休憩している場合は、すべてのJupyterタブを閉じてコンピューティングインスタンスを**停止**し、不要なコストが発生しないようにすることができます。
