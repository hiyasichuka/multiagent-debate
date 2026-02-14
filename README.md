# Multiagent Debate - LLMの精度は議論で上がるのか

論文 [Improving Factuality and Reasoning in Language Models through Multiagent Debate](https://arxiv.org/abs/2305.14325) (Du et al., 2023) の手法を、ローカルLLMで再現・検証するプロジェクトです。

## Multiagent Debate とは

LLMに問題を解かせるとき、1回だけ答えさせるよりも、複数のエージェントに互いの回答を見せ合って議論させた方が精度が上がる、という研究です。

仕組みは以下の通りです。

```
Round 1 ... 3つのエージェントが、それぞれ独立に問題を解く
Round 2 ... 各エージェントが他の2つの回答を読み、それを踏まえてもう一度解き直す
最終回答 ... 3つの回答を多数決で集約する
```

論文では ChatGPT (gpt-3.5-turbo-0301) を使い、算数・常識推論・読解など複数のベンチマークで精度向上を報告しています。

では、ローカルで動く軽量モデルでも同じ効果は得られるのかを確かめます。

## ベンチマーク

[GSM8K](https://github.com/openai/grade-school-math) (Grade School Math 8K) を使います。小学校レベルの算数文章題が約8,500問収録されており、LLMの数学的推論能力を測る標準的なベンチマークです。

問題の例を示します。

> Janet's ducks lay 16 eggs per day. She eats three for breakfast every morning and bakes muffins for her friends every day with four. She sells every duck egg at the farmers' market for $2. How much does she make every day at the farmers' market?
>
> 正解 ... 18

## セットアップ

### 前提条件

- macOS または Linux
- [Ollama](https://ollama.com/) がインストールされていること
- [uv](https://docs.astral.sh/uv/) がインストールされていること

### 1. リポジトリをクローン

```bash
git clone https://github.com/hiyasichuka/multiagent-debate.git
cd multiagent-debate
```

### 2. LLMモデルをダウンロード

Ollama でモデルを取得します。Notebook のデフォルトでは `qwen2.5:1.5b`（約1GB）を使用します。1.5B と軽量ながら GSM8K スコア 73.2 と同サイズ帯では数学的推論に強いモデルです（[Qwen2.5 公式ブログ](https://qwenlm.github.io/blog/qwen2.5-llm/)）。

```bash
ollama pull qwen2.5:1.5b
```

他のモデルを試したい場合は、以下のように追加で取得してください。

```bash
ollama pull phi3:mini      # 3.8B, 約2.3GB
ollama pull llama3.2:3b    # 3B, 約2GB
```

### 3. Python環境を構築

```bash
uv sync
```

仮想環境の作成と依存パッケージのインストールがまとめて行われます。

### 4. Jupyter Notebook を起動

```bash
uv run jupyter notebook
```

ブラウザが開いたら `notebooks/multiagent_debate_gsm8k.ipynb` を選択してください。

## Notebook の構成

| Cell | 内容 | 説明 |
|------|------|------|
| 1 | セットアップ | Ollama 接続確認、モデル選択 |
| 2 | データセット読み込み | GSM8K のダウンロードとサンプル数の設定（デフォルト20問） |
| 3 | コア関数の定義 | LLM 呼び出し、プロンプト生成、回答抽出、多数決 |
| 4 | ベースライン実験 | Single Agent（1体で1回だけ回答）の精度を測定 |
| 5 | Debate 実験 | 3 エージェント x 2 ラウンドの精度を測定 |
| 6 | 評価と可視化 | 精度の棒グラフ、問題ごとの正誤パターン分析 |
| 7 | 感度分析（オプション） | エージェント数やラウンド数を変えた比較 |

上から順に実行すれば結果が得られます。

## 実験結果（qwen2.5:1.5b, N=20）

| 手法 | 精度 | 標準誤差 |
|------|------|----------|
| Single Agent（ベースライン） | 45.0% (9/20) | 11.1% |
| Multiagent Debate（3 agents, 2 rounds） | 65.0% (13/20) | 10.7% |
| **差分** | **+20.0pt** | |

1.5B の軽量モデルでも Debate による精度向上が確認できました。ただし N=20 のため標準誤差が大きく、試行ごとのばらつきがあります。詳しい考察は [Zenn の解説記事](https://zenn.dev/hiyasichuka/articles/multiagent-debate-local-llm) にまとめています。

## 実験パラメータの変更

Notebook 内の変数を書き換えることで調整できます。

```python
# Cell 1 -- モデルを変更
MODEL = "phi3:mini"

# Cell 2 -- サンプル数を増やす（精度の信頼度が上がるが、実行時間も伸びる）
N_SAMPLES = 100

# Cell 5 -- エージェント数やラウンド数を変更
N_AGENTS = 5
N_ROUNDS = 3
```

## プロジェクト構成

```
multiagent-debate/
├── notebooks/
│   ├── multiagent_debate_gsm8k.ipynb   # メイン Notebook
│   └── results/                         # 実験結果の JSON（自動生成、gitignore 対象）
├── pyproject.toml                       # 依存管理 (uv)
└── README.md
```

## 参考文献

- Du, Y., Li, S., Torralba, A., Tenenbaum, J. B., & Mordatch, I. (2023). [Improving Factuality and Reasoning in Language Models through Multiagent Debate](https://arxiv.org/abs/2305.14325). *ICML 2024*.
- Cobbe, K., Kosaraju, V., Bavarian, M., et al. (2021). [Training Verifiers to Solve Math Word Problems](https://arxiv.org/abs/2110.14168). (GSM8K dataset)

## License

MIT
