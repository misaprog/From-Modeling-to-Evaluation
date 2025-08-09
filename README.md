# From-Modeling-to-Evaluation

### 目標

このラボを完了すると、以下のことができるようになります。

モデルを作成する

モデルを評価する

---

### 目次

1. まとめ
   
2. データモデリング
  
3. モデル評価

---
まとめ

ラボ「理解から準備へ」では、データを調査してモデリングの準備をしました。

このレシピデータは、Yong-Yeol Ahn という研究者によってまとめられたもので、次の 3 つの異なる Web サイトから数万件の料理レシピ (料理と材料) を収集しました。

---

## コードの解説

```python
import pandas as pd  # データをDataFrame形式で読み込むためのpandasライブラリ
pd.set_option("display.max_columns", None)  # DataFrame表示時に列を省略せずすべて表示する設定

import numpy as np  # 数値計算や配列操作のためのNumPyライブラリ

import re  # 正規表現（パターンマッチング）のためのreライブラリ

import random  # 乱数生成やランダムな処理のためのrandomライブラリ
```

### 各ライブラリの役割

| ライブラリ             | 用途           | 主な機能例                           |
| ----------------- | ------------ | ------------------------------- |
| **pandas** (`pd`) | データ分析・加工     | CSVやExcelファイルの読み込み、表形式データの集計・変換 |
| **NumPy** (`np`)  | 高速な数値計算      | 多次元配列、ベクトル・行列演算、統計計算            |
| **re**            | 正規表現による文字列処理 | パターン検索、置換、文字列の抽出                |
| **random**        | 乱数生成         | ランダムサンプリング、乱数によるシミュレーション        |

### 設定のポイント

* `pd.set_option("display.max_columns", None)`
  → pandasのDataFrameを表示するとき、列が多い場合に「…」と省略されるのを防ぎ、すべての列を表示します。
  データ分析時に全列を確認したいときに便利です。

---

## コードの解説

```python
from pyodide.http import pyfetch  # Pyodide環境でHTTPリクエストを行うためのモジュールをインポート

# 非同期関数を定義：指定したURLからファイルをダウンロードして保存する
async def download(url, filename):
    response = await pyfetch(url)  # URLにアクセスしてデータを取得
    if response.status == 200:     # HTTPステータスが200（成功）の場合のみ処理
        with open(filename, "wb") as f:  # ローカルファイルをバイナリ書き込みモードで開く
            f.write(await response.bytes())  # 取得したデータをファイルに書き込み

# ダウンロードするCSVファイルのURL
path = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DS0103EN-SkillsNetwork/labs/Module%202/recipes.csv"

# Pyodide環境でデータをダウンロード（ローカル実行の場合はコメントアウト）
await download(path, "recipes.csv")
```

---

### このコードの役割

1. **`pyfetch`**

   * Pyodide用のHTTPリクエスト関数で、Web上からデータを取得するために使用します。
   * 通常の`requests`ライブラリはPyodide環境では動かないため、代わりに使います。

2. **非同期処理（`async` / `await`）**

   * ネットワーク通信は時間がかかるため、非同期で処理を進めることで待機中も他の処理を行えるようにします。

3. **`if response.status == 200`**

   * ダウンロード成功（HTTPステータス200）を確認してからファイル保存する安全策です。

4. **`with open(..., "wb")`**

   * `"wb"`はバイナリモードで書き込みを行う設定。CSVなどの非テキストデータでも安全に保存できます。

---

💡 **補足**

* このコードは\*\*Pyodide環境（例: JupyterLiteやブラウザ上のPython実行環境）\*\*向けです。
* ローカルのJupyter Notebookや通常のPython環境では、`pyfetch`は使わず、代わりに`requests`や`urllib`を使用します。

---
そのエラーは、**`pyodide` がローカルのJupyter Notebook環境には存在しない**ことが原因です。

`pyodide` は **JupyterLite やブラウザ上のPython（Pyodide環境）** でしか使えないモジュールなので、ローカルPCで動かすと `ModuleNotFoundError` になります。

---

### 💡 ローカル実行用のコードに書き換える方法

ローカルや通常のJupyter Notebookでは、`requests` を使えば同じことができます。

```python
import requests

# ダウンロード先のURL
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DS0103EN-SkillsNetwork/labs/Module%202/recipes.csv"

# データを取得
response = requests.get(url)

# HTTPステータス200（成功）の場合のみ保存
if response.status_code == 200:
    with open("recipes.csv", "wb") as f:
        f.write(response.content)
    print("Download completed: recipes.csv")
else:
    print("Failed to download file. Status code:", response.status_code)
```

---

### 違い

* **Pyodide版** → ブラウザ上（JupyterLiteやCourseraのノートブックなど）で動く。
* **Requests版** → ローカルPCやAnaconda環境のJupyter Notebookで動く。

---
便宜上、すでにデータは IBM サーバー上に配置されているため、サーバーからデータをダウンロードし、recipes というデータフレームにデータを読み込みます。

---

```python
recipes = pd.read_csv("recipes.csv")
```

* `pd.read_csv()` はPandasでCSVファイルを読み込む関数
* `"recipes.csv"` は、さっきダウンロードしたファイル名
* 読み込み結果は `recipes` という変数に保存されます（これがDataFrameになります）

```python
print("Data read into dataframe!")
```

* 単純に読み込み完了のメッセージを表示
* コメントの `# takes about 30 seconds` は、環境によっては少し時間がかかることを示しています

---

## **① Jupyter Notebook用（コード内コメント付き）**

```python
# --- 列名の修正 ---
# CSVの最初の列名が "cuisine" ではない可能性があるので、正しい名前に変更
column_names = recipes.columns.values
column_names[0] = "cuisine"
recipes.columns = column_names

# --- 料理名を小文字に変換 ---
recipes["cuisine"] = recipes["cuisine"].str.lower()

# --- 料理名を統一する ---
# 国名や地域名が混在しているので、表記を統一
recipes.loc[recipes["cuisine"] == "austria", "cuisine"] = "austrian"
recipes.loc[recipes["cuisine"] == "belgium", "cuisine"] = "belgian"
recipes.loc[recipes["cuisine"] == "china", "cuisine"] = "chinese"
recipes.loc[recipes["cuisine"] == "canada", "cuisine"] = "canadian"
recipes.loc[recipes["cuisine"] == "netherlands", "cuisine"] = "dutch"
recipes.loc[recipes["cuisine"] == "france", "cuisine"] = "french"
recipes.loc[recipes["cuisine"] == "germany", "cuisine"] = "german"
recipes.loc[recipes["cuisine"] == "india", "cuisine"] = "indian"
recipes.loc[recipes["cuisine"] == "indonesia", "cuisine"] = "indonesian"
recipes.loc[recipes["cuisine"] == "iran", "cuisine"] = "iranian"
recipes.loc[recipes["cuisine"] == "italy", "cuisine"] = "italian"
recipes.loc[recipes["cuisine"] == "japan", "cuisine"] = "japanese"
recipes.loc[recipes["cuisine"] == "israel", "cuisine"] = "jewish"
recipes.loc[recipes["cuisine"] == "korea", "cuisine"] = "korean"
recipes.loc[recipes["cuisine"] == "lebanon", "cuisine"] = "lebanese"
recipes.loc[recipes["cuisine"] == "malaysia", "cuisine"] = "malaysian"
recipes.loc[recipes["cuisine"] == "mexico", "cuisine"] = "mexican"
recipes.loc[recipes["cuisine"] == "pakistan", "cuisine"] = "pakistani"
recipes.loc[recipes["cuisine"] == "philippines", "cuisine"] = "philippine"
recipes.loc[recipes["cuisine"] == "scandinavia", "cuisine"] = "scandinavian"
recipes.loc[recipes["cuisine"] == "spain", "cuisine"] = "spanish_portuguese"
recipes.loc[recipes["cuisine"] == "portugal", "cuisine"] = "spanish_portuguese"
recipes.loc[recipes["cuisine"] == "switzerland", "cuisine"] = "swiss"
recipes.loc[recipes["cuisine"] == "thailand", "cuisine"] = "thai"
recipes.loc[recipes["cuisine"] == "turkey", "cuisine"] = "turkish"
recipes.loc[recipes["cuisine"] == "vietnam", "cuisine"] = "vietnamese"
recipes.loc[recipes["cuisine"] == "uk-and-ireland", "cuisine"] = "uk-and-irish"
recipes.loc[recipes["cuisine"] == "irish", "cuisine"] = "uk-and-irish"

# --- 50件未満のレシピは除外 ---
recipes_counts = recipes["cuisine"].value_counts()
cuisines_indices = recipes_counts > 50
cuisines_to_keep = list(np.array(recipes_counts.index.values)[np.array(cuisines_indices)])
recipes = recipes.loc[recipes["cuisine"].isin(cuisines_to_keep)]

# --- Yes/No を 1/0 に変換 ---
recipes = recipes.replace(to_replace="Yes", value=1)
recipes = recipes.replace(to_replace="No", value=0)
```

---

## **② GitHub README用（文章説明）**

**データ前処理の流れ**

1. **列名の修正**

   * CSVの最初の列を `cuisine` に統一し、後の分析で使いやすくします。

2. **文字の小文字化**

   * 料理名（cuisine列）の文字をすべて小文字に変換し、大文字・小文字の違いによるデータの不一致を防ぎます。

3. **料理名の統一**

   * 国名や地域名が混在していたため、統一した表記（例：`austria` → `austrian`、`spain` / `portugal` → `spanish_portuguese`）に変換しました。

4. **少数データの除外**

   * レシピ数が50件未満の料理カテゴリーを除外し、モデル学習に必要なデータの安定性を確保します。

5. **Yes/No の数値変換**

   * 材料の有無を表す `Yes`/`No` を、それぞれ `1`/`0` に変換し、機械学習で使える形式にしました。

---
## データモデリング

次に、決定木を構築するために、さらに多くのライブラリと依存関係をダウンロードしてインストールします。

---

## **コード解説**

```python
# 決定木モデルの構築に必要なライブラリをインポート

from sklearn import tree                     # scikit-learn の決定木アルゴリズム
from sklearn.metrics import accuracy_score, confusion_matrix  # 精度評価用
import matplotlib.pyplot as plt               # グラフ描画ライブラリ

# graphviz（決定木をきれいに可視化する外部ツール）も利用可能
# ローカル環境なら以下のようにインストールして使用可能
# !conda install python-graphviz --yes
# from sklearn.tree import export_graphviz

import itertools                              # 混同行列の可視化などでループ処理に利用
```

### **ポイント解説**

1. **`from sklearn import tree`**

   * scikit-learn に含まれる **決定木アルゴリズム**（分類や回帰に利用可能）を読み込みます。

2. **`accuracy_score`, `confusion_matrix`**

   * **accuracy\_score**：モデルがどれくらい正しく予測できたかを計算します（正解率）。
   * **confusion\_matrix**：実際の値と予測値の比較表（分類モデルの性能評価に使用）。

3. **`matplotlib.pyplot as plt`**

   * データやモデルの結果を可視化するためのグラフ描画ライブラリです。

4. **Graphviz 関連（コメントアウト部分）**

   * Graphviz は決定木をわかりやすい図で表示できる外部ツール。
   * ローカル環境で `conda install python-graphviz` すると `export_graphviz` 関数で利用可能。

5. **`import itertools`**

   * 繰り返し処理や組み合わせ処理を簡単に行えるPythonの標準ライブラリ。
   * 特に混同行列のプロット時に座標の組み合わせ生成に使われます。

---
#### [bamboo_tree] アジア料理とインド料理のみ

ここでは、アジア料理（韓国料理、日本料理、中国料理、タイ料理）とインド料理のレシピのみを対象とした決定木を作成しています。この操作を行う理由は、データが特定の料理または複数の料理グループ（今回の場合はアメリカ料理）に偏っている場合、決定木がうまく動作しないためです。アメリカ料理を分析から除外するか、データの異なるサブセットのみを対象に決定木を構築することもできます。ここでは2つ目の解決策を採用します。

アジア料理とインド料理に関するデータを使用して決定木を作成し、決定木に bamboo_tree という名前を付けます。

---
### アジア・インド料理のサブセット抽出と決定木モデルの学習

- `recipes` データフレームから、韓国、日本、中国、タイ、インドの5つの料理カテゴリーだけを抽出しています。
- `cuisines` は料理のラベル（ターゲット変数）で、`ingredients` は料理の材料情報（特徴量）です。
- `DecisionTreeClassifier` クラスを使い、深さを3に制限した決定木モデルを作成しています。
- `fit` メソッドで材料データを入力し、対応する料理名を学習させています。

```python
asian_indian_recipes = recipes[recipes.cuisine.isin(["korean", "japanese", "chinese", "thai", "indian"])]
cuisines = asian_indian_recipes["cuisine"]
ingredients = asian_indian_recipes.iloc[:,1:]

bamboo_tree = tree.DecisionTreeClassifier(max_depth=3)
bamboo_tree.fit(ingredients, cuisines)

print("Decision tree model saved to bamboo_tree!")

```
### 決定木をプロットし、決定木がどのように見えるかを確認しましょう。

### 決定木の可視化

- `graphviz` ライブラリを使うと詳細な決定木を画像ファイルとして出力できますが、ここではscikit-learnの `plot_tree` メソッドを利用しています。
- `plot_tree` では、特徴量名やクラス名を表示し、ノードを色分け、ノードIDを表示してわかりやすく可視化しています。
- `plt.figure(figsize=(40,20))` で図のサイズを指定し、見やすい大きさに調整しています。

```python
# graphvizでの可視化はコメントアウト
# export_graphviz(bamboo_tree,
#                 feature_names=list(ingredients.columns.values),
#                 out_file="bamboo_tree.dot",
#                 class_names=np.unique(cuisines),
#                 filled=True,
#                 node_ids=True,
#                 special_characters=True,
#                 impurity=False,
#                 label="all",
#                 leaves_parallel=False)

# with open("bamboo_tree.dot") as bamboo_tree_image:
#     bamboo_tree_graph = bamboo_tree_image.read()
# graphviz.Source(bamboo_tree_graph)

plt.figure(figsize=(40,20))
_ = tree.plot_tree(
    bamboo_tree,
    feature_names=list(ingredients.columns.values),
    class_names=np.unique(cuisines),
    filled=True,
    node_ids=True,
    impurity=False,
    label="all",
    fontsize=20,
    rounded=True
)
plt.show()

```
#### 決定木は次のことを学習しました。

レシピにクミンと魚は含まれていて、ヨーグルトは含まれていない場合、それはタイ料理のレシピである可能性が高いです。

レシピにクミンは含まれていて、魚と醤油は含まれていない場合、それはインド料理のレシピである可能性が高いです。

ツリーの残りの枝を分析して、さまざまなレシピの料理を決定するための同様のルールを導き出すことができます。

別の料理のサブセットを選択して、それらのレシピの決定木を構築することもできます。ヨーロッパ料理をいくつか選択して決定木を構築し、それらの料理を区別する材料を探索することもできます。

---
次に、アジア料理とインド料理のモデルを評価するために、データセットをトレーニングセットとテストセットに分割します。

トレーニングセットを用いて決定木を構築します。次に、テストセットでモデルをテストし、モデルが予測した料理と実際の料理を比較します。

まず、アジア料理とインド料理に関するデータのみを含む新しいデータフレームを作成し、「bamboo」という名前を付けます。

---
```
bamboo = recipes[recipes.cuisine.isin(["korean", "japanese", "chinese", "thai", "indian"])]

```

#### 説明

- `recipes`：料理データのPandas DataFrame。
- `recipes.cuisine.isin([...])`：`cuisine`列の値がリスト内のいずれかに該当するかを判定する条件式。
- `recipes[...]`：条件に合致する行のみを抽出。

このコードは、アジアの代表的な5カ国（韓国、日本、中国、タイ、インド）の料理データのみを抽出して `bamboo` に格納しています。

---
### 次に、各料理にいくつのレシピが存在するかを確認します。
---
```
bamboo["cuisine"].value_counts()
```
#### 説明

- `bamboo["cuisine"]`：抽出済みのDataFrame `bamboo` の `cuisine` 列。
- `.value_counts()`：各料理ジャンルの出現頻度（件数）を降順で返すメソッド。

この処理は、指定したアジア料理（韓国、日本、中国、タイ、インド）の各ジャンルが、データ内に何件存在するかを集計しています。

---

### 各料理から 30 個のレシピを削除してテスト セットとして使用し、このテスト セットに bamboo_test という名前を付けます。
```
# set sample size
sample_n = 30

```
#### 説明

- `sample_n = 30` は、後続の処理で使用するサンプルのサイズ（抽出件数）を30に設定するための変数定義です。
- 例えば、データの一部をランダムに抽出する際などに利用します。

---
### 次に、各料理からランダムに選ばれた30個のレシピを含むデータフレームを作成します。
 
#### ポイント

- `groupby("cuisine")` で料理ジャンルごとにデータをグループ化  
- `sample(n=30, random_state=1234)` で各グループから30件をランダム抽出（シード固定で再現可能）  
- `group_keys=False` によりグループ名（`cuisine`）をインデックスに含めない  
- 抽出結果を `reset_index(drop=True)` でインデックスを振り直し  

#### 分割処理

- 抽出後のデータは材料情報とラベル（ジャンル名）を含むため、  
  `.drop(columns=["cuisine"])` で材料部分のみのデータフレームを作成  
- `.loc[:, "cuisine"]` で対応するジャンルラベルだけのシリーズを取得  

#### コード例

```python
sample_n = 30

bamboo_test = bamboo.groupby("cuisine", group_keys=False)\
                    .sample(n=sample_n, random_state=1234)\
                    .reset_index(drop=True)

bamboo_test_ingredients = bamboo_test.drop(columns=["cuisine"])  # 材料データ
bamboo_test_cuisines = bamboo_test["cuisine"]                   # 料理ジャンルラベル

```
### 各料理に30種類のレシピがあることを確認します。

#### 解説

bamboo_test は各料理ジャンル（cuisine）ごとに30件ずつランダムに抽出したデータフレームです。

bamboo_test["cuisine"] で、そのデータの中の「料理ジャンル」列だけを取り出します。

.value_counts() は、その列に含まれる値（ここでは料理ジャンル名）がそれぞれいくつあるかをカウントして、
各ジャンルごとの件数を降順にまとめて表示します。

つまり
このコードは、「本当に各ジャンルから30件ずつ抽出できているか？」を確認するために使います。

表示結果で、すべてのジャンルの値が30になっていれば、期待通りにサンプリングできていることがわかります。

