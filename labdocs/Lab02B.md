# Lab 2B: Azure ML Designerを使用してサービスをデプロイする

トレーニング済みのモデルができたので、トレーニングパイプラインを使用して、新しいデータをスコアリングするための推論パイプラインを作成するために使用できます。

> **Note**: この記事の執筆時点では、Azure Machine Learning Designerは*プレビュー*状態です。予期しないエラーが発生する場合があります。

## 始める前に

このラボを開始する前に、[Lab1A](Lab01A.md)および[Lab1B](Lab01B.md)を完了していることを確認してください。これらには、このラボで使用するAzure Machine Learningワークスペースおよびその他のリソースを作成するタスクが含まれています。 [Lab2A](Lab02A.md)も完了する必要があります。これには、このラボで使用するDesignerトレーニングパイプラインを作成するタスクが含まれています。

## Task 1: Computeの準備

このラボでは、推論パイプラインをAzure Kubernetes Service（AKS）クラスターのコンテナー化されたサービスとして公開します。 AKSクラスターは初期化に時間がかかることがあるため、推論パイプラインを準備する前にプロセスを開始します。

1. [Azure Machine Learning studio](https://ml.azure.com)のワークスペースの[**Compute**]ページで、各タブの下にある既存の計算ターゲットを確認します。これらには以下が含まれます。
    * **Compute Instances**: 前のラボで作成したコンピューティングインスタンス。
    * **Training Clusters**: 前のラボで作成した**aml-cluster**計算ターゲット]。
    * **Inference Clusters**: なし（まだ！）
    * **Attached Compute**: なし（これは、ワークスペース外に存在する仮想マシンまたはDatabricksクラスターを接続できる場所です）

2. [**Compute Instances**]タブで、計算インスタンスがまだ実行されていない場合は起動します。この実習ラボで後で使用します。

3. [**Inference Clusters**]タブで、次の設定で新しいクラスターを追加します:
    * **Compute name**: aks-cluster
    * **Kubernetes Service**: Create new
    * **Region**: *Any available region*
    * **Virtual Machine size**: Standard_F2s_v2
    * **Cluster purpose**: Dev-test
    * **Number of nodes**: 3
    * **Network configuration**: Basic
    * **Enable SSL configuration**: Unselected

4. 計算ターゲットが*Creating*状態にあることを確認し、次のタスクに進みます。

## Task 2: 推論パイプラインを作成する

推論計算のプロビジョニング中に、展開用の推論パイプラインを準備できます。

1. **Designer**ページで、前のラボで作成した**Visual Diabetes Training**パイプラインを開きます。
2. [**Create inference pipeline**]ドロップダウンリストで、[**Real-time inference pipeline**]をクリックします。数秒後、**Visual Diabetes Training-real time inference**という名前のパイプラインの新しいバージョンが開きます。

3. 新しいパイプラインの名前を**Predict Diabetes**に変更してから、新しいパイプラインを確認します。変換データとトレーニングステップの一部はこのパイプラインにカプセル化されているため、トレーニングデータの統計を使用して新しいデータ値を正規化し、トレーニングされたモデルを使用して新しいデータをスコアリングします。
4. 推論パイプラインは、新しいデータが元のトレーニングデータのスキーマと一致すると想定しているため、トレーニングパイプラインの**diabetes dataset**モジュールが含まれています。ただし、この入力データには、モデルが予測する**Diabetic**ラベルが含まれています。これは、糖尿病の予測がまだ行われていない新しい患者データに含めるのは直感的ではありません。このモジュールを削除し、**Apply Transformation**モジュールの同じ**dataset**入力に接続されている**Data Input and Output**セクションから**Enter Data Manually**モジュールに置き換えます。 **Web Service Input**。次に、**Enter Data Manually**モジュールの設定を変更して、次のCSV入力を使用します。これには、3つの新しい患者観察のラベルなしの特徴値が含まれます:

    ```CSV
    PatientID,Pregnancies,PlasmaGlucose,DiastolicBloodPressure,TricepsThickness,SerumInsulin,BMI,DiabetesPedigree,Age
    1882185,9,104,51,7,24,27.36983156,1.350472047,43
    1662484,6,73,61,35,24,18.74367404,1.074147566,75
    1228510,4,115,50,29,243,34.69215364,0.741159926,59
    ```

5. 推論パイプラインには**Evaluate Model**モジュールが含まれていますが、これは新しいデータから予測する場合には役に立たないため、このモジュールを削除してください。
6. **Score Model**モジュールの出力には、入力フィーチャのすべてと、予測ラベルと確率スコアが含まれます。出力を予測と確率のみに制限するには、**Score Model**モジュールと**Web Service Output**間の接続を削除し、**Data Transformations** **Apply SQL Transformation **モジュールを追加します。セクション、**Score Model**モジュールからの出力を**Apply SQL Transformation**の**t1**（左端）入力に接続し、**Apply SQL Transformation** の出力を接続します**Web Service Output**のモジュール。次に、**Apply SQL Transformation**モジュールの設定を変更して、次のSQLクエリスクリプトを使用します。:

    ```SQL
    SELECT PatientID,
           [Scored Labels] AS DiabetesPrediction,
           [Scored Probabilities] AS Probability
    FROM t1
    ```

7. パイプラインが次のように見えることを確認します:

    ![Visual Inference Pipeline](images/visual-inference.jpg)

8. トレーニングに使用した**aml-compute**計算ターゲットで、**predict-diabetes**という名前の新しい実験としてパイプラインを実行します。これは時間がかかる場合があります！

## Task 3: Webサービスを公開する

これで、リアルタイム推論用の推論パイプラインができました。これは、クライアントアプリケーションが使用するWebサービスとして公開できます。

1. **Compute**ページに戻り、**Inference Compute**タブでビューを更新し、**aks-cluster**コンピューティングが作成されたことを確認します。そうでない場合は、推論クラスターが作成されるのを待ちます。これにはかなり時間がかかる場合があります。

2. **Designer**タブに戻り、**Predict Diabetes**推論パイプラインを再度開きます。まだ実行が終了していない場合は、完了するまで待ちます。次に、**Apply SQL Transformation**モジュールの出力を視覚化して、入力データ内の3つの患者の観察の予測ラベルと確率を確認します。

3. 右上の[**Deploy**]をクリックし、作成した**aks-cluster**コンピューティングターゲットに**predict-diabetes**という名前の新しいリアルタイムエンドポイントを設定します。
4. Webサービスがデプロイされるのを待ちます-これには数分かかる場合があります。展開ステータスは、Designerインターフェイスの左上に表示されます。

    > **Tip**: サービスがデプロイされるのを待っている間に、Azure Machine Learning Designerのドキュメントをしばらく見てください。[https://docs.microsoft.com/azure/machine-learning/service/concept-designer](https://docs.microsoft.com/azure/machine-learning/service/concept-designer)

## Task 4: Webサービスをテストする

これで、クライアントアプリケーションからデプロイされたサービスをテストできます。この場合、ノートブックVMでノートブックを使用します。

1. [**エンドポイント(Endpoint)**]ページで、**predict-diabetes**リアルタイムエンドポイントを開きます。
2. **predict-diabetes**エンドポイントが開いたら、**テスト(Test)**ページで、デフォルトのテスト入力パラメーターを書き留めてから**Test**をクリックして、デプロイされたWebサービスに送信し、予測を生成します。
3. [**使用(Consume)**]タブで、**Python**に提供されているサンプルコードを表示し、Pythonサンプルスクリプト全体をクリップボードにコピーします。
4. [**Compute**]ページで、コンピューティングインスタンスがまだ実行されていない場合は、起動するまで待ちます。次に、**Jupyter**リンクをクリックします。
5. Jupyterの**Users/DP100**フォルダーで、**02B-Visual Designer.ipynb**を使用して開きます。
6. ノートブックで、コピーしたコードを空のコードセルに貼り付けます。
7. コードセルを実行し、Webサービスから返された出力を表示します。

## Task 5: Webサービスを削除して計算する

WebサービスはKubernetesクラスターでホストされます。さらに試してみるつもりがない場合は、エンドポイントとクラスターを削除して、不要なAzure料金が発生しないようにする必要があります。

1. Azure MLワークスペースの*Studio* Webインターフェイスの[**Endpoints**]タブで、**predict-diabetes**エンドポイントを選択します。次に、**削除**（🗑）ボタンをクリックして、エンドポイントを削除することを確認します。
2. [**Compute**]ページの[**Inference Clusters**]タブで、[**aks-cluster**]エンドポイントを選択します。次に、**削除**（🗑）ボタンをクリックして、計算ターゲットを削除することを確認します。

> **Note**: [次の演習](Lab03A.md)に直接進む場合は、コンピューティングインスタンスを実行したままにします。休憩する場合は、Jupyterタブを閉じてコンピューティングインスタンスを**停止**し、不要なコストが発生しないようにすることができます。

