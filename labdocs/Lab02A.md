# Lab 2A: Azure ML Designerを使用してトレーニングパイプラインを作成する

*Designer*インターフェイスは、機械学習モデルを作成するためのワークフロー、またはデータの取り込み、変換、モデルトレーニングモジュールの*パイプライン*を定義できるドラッグアンドドロップ環境を提供します。その後、このパイプラインを、クライアントアプリケーションが*推論*（新しいデータから予測を生成する）に使用できるWebサービスとして公開できます。

> **Note**: この記事の執筆時点では、Azure Machine Learning Designerは*プレビュー*状態です。予期しないエラーが発生する場合があります。

## 始める前に

このラボを開始する前に、[ラボ1A](Lab01A.md)および[ラボ1B](Lab01B.md)を完了していることを確認してください。これらには、このラボで使用するAzure Machine Learningワークスペースおよびその他のリソースを作成するタスクが含まれています。

## Task 1: デザイナーパイプラインを作成してデータを探索する

Designerを開始するには、最初にパイプラインを作成し、使用するデータセットを追加する必要があります。

1. ワークスペースの[Azure Machine Learning studio](https://ml.azure.com)で、**Designer**ページを表示し、新しいパイプラインを作成します。
2. [**設定**]ペインで、デフォルトのパイプライン名（**Pipeline-Created-on-date**）を**Visual Diabetes Training**（**設定**ペインが表示されていない場合、上部のパイプライン名の横にある**⚙**アイコンをクリックします）。
3. パイプラインを実行する計算ターゲットを指定する必要があることに注意してください。 [**設定**]ペインで[**コンピューティングターゲットの選択**]をクリックし、前のラボで作成した**aml-cluster**コンピューティングターゲットを選択します。
4. デザイナーの左側で、**データセット**セクションを展開し、前の演習で作成した**diabetes dataset**データセットをキャンバスにドラッグします。
5. キャンバスで**diabetes dataset**モジュールを選択し、その設定を表示します。 [**output**]タブで、[**visualize**]アイコン（縦棒グラフに似ています）をクリックします。
6. データのスキーマを確認し、さまざまな列の分布をヒストグラムとして表示できることに注意してください。次に、視覚エフェクトを閉じます。

## Task 2: データ変換を追加

モデルをトレーニングする前に、通常、いくつかの前処理変換をデータに適用する必要があります。

1. 左側のペインで、**Data Transformation**セクションを展開します。このセクションには、モデルのトレーニングの前にデータを変換して前処理するために使用できる幅広いモジュールが含まれています。 **Normalize Data**モジュールをキャンバスの**diabetes dataset**モジュールの下にドラッグします。次に、**diabetes dataset**モジュールの出力を**Normalize Data**モジュールの入力に接続します。
2. **Normalize Data**モジュールを選択し、その設定を表示します。変換方法と変換する列を指定する必要があることに注意してください。次に、変換を**ZScore**のままにして、以下を含むように列を編集します
column names:
    * PlasmaGlucose
    * DiastolicBloodPressure
    * TricepsThickness
    * SerumInsulin
    * BMI
    * DiabetesPedigree

    > **Note**: 数値列を正規化して同じスケールにし、値が大きい列がモデルトレーニングを支配しないようにします。通常、このような前処理変換の全体を適用してトレーニング用のデータを準備しますが、この演習では簡単に説明します。

3. これで、トレーニングと検証のためにデータを個別のデータセットに分割する準備ができました。左側のペインの**Data Transformations**セクションで、**Split Data**モジュールを**Normalize Data**モジュールの下のキャンバスにドラッグします。次に、**Normalize Data**モジュールの*Transformed Dataset*（左）出力を**Split Data**モジュールの入力に接続します。

4. **Split Data**モジュールを選択し、次のように設定します:
    * **Splitting mode** Split Rows
    * **Fraction of rows in the first output dataset**: 0.7
    * **Random seed**: 123
    * **Stratified split**: False

## Task 3: モデルトレーニングモジュールを追加する

データを準備し、トレーニングデータセットと検証データセットに分割したら、パイプラインを構成してモデルをトレーニングおよび評価できます。

1. Expand the **Model Training** section in the pane on the left, and drag a **Train Model** module to the canvas, under the **Split Data** module. Then connect the *Result dataset1* (left) output of the **Split Data** module to the *Dataset* (right) input of the **Train Model** module.
1. 左側のペインの**Model Training**セクションを展開し、**Train Model**モジュールをキャンバスの**Split Data**モジュールの下にドラッグします。次に、**Split Data**モジュールの*Result dataset1*（左）出力を**Train Model**モジュールの*Dataset*（右）入力に接続します。

2. トレーニング中のモデルは**Diabetic**値を予測するため、**Train Model**モジュールを選択し、設定を変更して**Label列**を**Diabetic**に設定します（大文字と小文字の区別！）

3. モデルが予測する**Diabetic**ラベルはバイナリ列（糖尿病患者の場合は1、糖尿病のない患者の場合は0）であるため、*classification*アルゴリズムを使用してモデルをトレーニングする必要があります。 **Machine Learning Algorithms**セクションを展開し、**Classification**の下で、**Two-Class Logistic Regression**モジュールをキャンバスの**Split Data**モジュールの左上にドラッグします。 **Train Model**モジュール。次に、その出力を**Train Model**モジュールの**Untrained model**（左）入力に接続します。

4. トレーニング済みのモデルをテストするには、元のデータを分割するときに保持した検証データセットをスコアリングするために使用する必要があります。 **Model Scoring＆Evaluation**セクションを展開し、**Score Model**モジュールをキャンバスの**Train Model**モジュールの下にドラッグします。次に、**Train Model**モジュールの出力を**Score Model**モジュールの**Trained Model**（左）に接続します。 **Results dataset2**（右）**Split Data**モジュールの出力を、**Score Model**モジュールの**Dataset**（右）入力にドラッグします。

5. モデルのパフォーマンスを評価するには、検証データセットのスコアリングによって生成されたいくつかのメトリックを調べる必要があります。 **Model Scoring＆Evaluation**セクションから、**Evaluate Model**モジュールを**Score Model**モジュールの下のキャンバスにドラッグし、**Score Model**モジュールの出力を**Score dataset**（左）**Evaluate Model**モジュールの入力.

## Task 4:  トレーニングパイプラインを実行する

データフローのステップを定義したら、トレーニングパイプラインを実行してモデルをトレーニングする準備ができました。

1. パイプラインが次のようになっていることを確認します（画像の各モジュールに、実行内容を記録するコメントが含まれていることに注意してください。実際のプロジェクトでDesignerを使用している場合、これを行うのは悪くありません！）:

    ![Visual Training Pipeline](images/visual-training.jpg)

2. 右上の[**実行(Run)**]をクリックします。次に、プロンプトが表示されたら、**visual-training**という名前の新しい*experiment*を作成して実行します。これにより、計算ターゲットが初期化され、パイプラインが実行されます。これには10分以上かかる場合があります。デザインキャンバスの右上にパイプライン実行のステータスが表示されます.

    > **Tip**: 実行中に、**Pipelines**および**Experiments**ページで作成されたパイプラインと実験を表示できます。完了したら、**Designer**ページの**Visual Diabetes Training**パイプラインに戻ります。

3. **Normalize Data**モジュールが終了したら（&#x2705;アイコンで示されます）、それを選択し、**Settings**ペインの**Outputs**タブの**Transformed dataset**セクションで、**VIsualize**アイコンをクリックし、変換された列の統計と分布の視覚化を表示できることに注意してください。


4. **Normalize Data**ビジュアライゼーションを閉じて、残りのモジュールが完了するのを待ちます。次に、**Evaluate Model**モジュールを視覚化して、モデルのパフォーマンスメトリックを確認します。

    > **Note**: このモデルのパフォーマンスはそれほど優れているわけではありません。一部には、最小限の機能エンジニアリングと前処理のみを実行したためです。いくつかの異なる分類アルゴリズムを試して結果を比較することができます（**Split Data**モジュールの出力を複数の**Train Model**および**Score Model**モジュールに接続できます。モデルを**Evaluate Model**モジュールに追加して、並べて比較します）。演習のポイントは、完璧なモデルをトレーニングするのではなく、単にDesignerインターフェースを紹介することです！
