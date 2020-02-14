# Lab 9A: Automated Machine Learningで変数の影響度を説明する

自動機械学習を使用してモデルをトレーニングする場合、変数の影響度の説明を生成するように実験を構成できます。

## 始める前に

このラボを開始する前に、[Lab1A](Lab01A.md)および[Lab1B](Lab01B.md)を完了していることを確認してください。これらには、このラボで使用するAzure Machine Learningワークスペースおよびその他のリソースを作成するタスクが含まれています。

## Task 1: Automated Machine Learningモデルの変数の影響度を確認する

このタスクでは、自動化された機械学習実験によって生成された変数の影響度を調べます。

1. [Azure Machine Learning studio](https://ml.azure.com)で、ワークスペースの**Compute**ページを表示します。 [**Compute Instances**]タブで、コンピューティングインスタンスが実行されていることを確認します。そうでない場合は、開始します。
2. コンピューティングインスタンスの実行中に、**Jupyter**リンクをクリックして、新しいブラウザータブでJupyterホームページを開きます。
3. Jupyterホームページの**Users/DP100**フォルダーで**09A-Reviewing Automated Machine Learning Explanations.ipynb**ノートブックを開きます。次に、ノートブックのメモを読み、各コードセルを順番に実行します。

> **Note**: [次の演習](Lab09B.md)に直接進む場合は、コンピューティングインスタンスを実行したままにします。休憩している場合は、すべてのJupyterタブを閉じてコンピューティングインスタンスを**停止**し、不要なコストが発生しないようにすることができます。
