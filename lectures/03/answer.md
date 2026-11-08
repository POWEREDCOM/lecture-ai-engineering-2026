# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
- `--trace` で見えたツール呼び出し（ツール名と引数）：
- 1つの質問での LLM 呼び出し回数（`requests`）：
- 追加したツールと、docstring に書いた説明：

## 宿題

- 取り組んだレベル：基礎
- やったこと：


###① `app.ask` と `app.agent` の比較
第1回と同じ質問を `app.ask` と `app.agent` に与え、回答の違いを確認した。
`app.ask` では、モデルが自身の知識だけを使って回答するため、講座固有の内容について一般論から推測した回答になる場合があった。一方、`app.agent` では用語集を検索してから回答するため、講座固有の情報を根拠として回答できた。
この比較から、モデル単体で回答する場合と、外部情報を検索して回答するエージェントでは、回答の根拠や正確性に違いが出ることを確認した。


###② `INSTRUCTIONS` と tool の docstring を変更したときの挙動比較
「LoRAとQLoRAの違いは？」という同じ質問に対して、以下の3条件で `app.agent --trace` を実行し、ツールの使われ方を比較した。
1. 変更なし
2. `app/agent.py` の `INSTRUCTIONS` のみ変更
3. `app/tools.py` の `search_glossary` のdocstringのみ変更
変更なしの状態では、LoRAとQLoRAをそれぞれ個別に検索し、`search_glossary` が2回呼ばれた。
`INSTRUCTIONS` のみ変更した場合は、回答文に多少の違いはあったものの、LoRAとQLoRAをそれぞれ検索する挙動は変わらず、tool_callsも2回のままだった。
一方、`tools.py` のdocstringを変更し、複数用語をまとめてqueryに指定できるようにしたところ、`search_glossary({'query': 'LoRA QLoRA'})` として1回の検索に変化した。tool_callsも2回から1回となり、ツールの説明文がモデルのツール利用方法に直接影響することを確認できた。


###③ `make_quiz` ツールの作成と動作確認
`app/tools.py` に `make_quiz(lecture: int) -> str` を追加し、指定した講義回の用語集から1つの用語を選んで確認問題と答えを返すようにした。
また、`app/agent.py` に `agent.tool_plain(tools.make_quiz)` を追加し、エージェントから `make_quiz` を利用できるように登録した。
その後、「第2回の内容から確認問題を1問出して」と指示して動作を確認したところ、モデルが `make_quiz` を `lecture=2` で呼び出し、第2回の用語集から問題を生成できた。実行するたびに異なる用語が選ばれることも確認した。
最後にテストを実行し、17件すべてのテストが成功したことを確認した。これにより、モデルがユーザーの指示に応じて適切なツールを選択し、その返り値を利用して回答を生成する流れを確認できた。


- 結果：

############①############
app.agent
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent "AIE2026の講座の宿題にある「3レベル」とは何ですか？"
[model] gemini-3.5-flash-lite
「3レベル」とは、AIE2026の宿題の構成についての仕組みです。

宿題には以下の3つの段階（レベル）が用意されており、受講生はどれか1つに取り組めばその回の宿題は完了となります（原則として採点は行われません）。

1. **基礎**：講義内容の確認
2. **実装**：自分でコードを書く
3. **発展**：調査や試行

関連する回：第1回
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


app.ask
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.ask "AIE2026の講座の宿題にある「3レベル」とは何ですか？"
[model] gemini-3.5-flash-lite
[質問] AIE2026の講座の宿題にある「3レベル」とは何ですか？

[usage] input_tokens=21 output_tokens=609
AIE2026（おそらくAIに関する特定の講座やプログラム）の宿題における「3レベル」という言葉ですね。

一般的なAI活用やプログラミング、あるいはその講座の文脈によって具体的に何を指すかは多少異なりますが、教育プログラムでよく使われる**「3つの習熟度レベル」**または**「3つの思考・実装レベル」**を指している可能性が高いです。

具体的には、以下のような分類のいずれか（または組み合わせ）に該当することがほとんどです。

---

### よくある「3つのレベル」の分類

#### 1. AI活用の習熟度レベル（よくあるパターンA）
AIツール（ChatGPTやClaudeなど）をどの程度使いこなせているかを見る基準です。
* **レベル1（基礎・入門）：** AIに言われた通りの質問をし、返ってきた答えをそのまま使う（コピペ）。
* **レベル2（応用・協働）：** プロンプトを工夫し、自分の目的に合わせてAIの出力を調整・編集する（壁打ち相手として使う）。
* **レベル3（高度・自動化）：** AIに役割（ペルソナ）を与えたり、APIやワークフローに組み込んで自分の分身のように自動化・高度活用する。

#### 2. 課題・実装の難易度レベル（よくあるパターンB）
宿題そのものの難易度が3段階に分かれている場合です。
* **レベル1（Basic）：** 必須課題。全員が必ずクリアすべき基本のステップ。
* **レベル2（Intermediate）：** 応用課題。基本に少し工夫や発展を加える内容。
* **レベル3（Advanced）：** 発展・挑戦課題。より高度なスキルが必要な、意欲的な人向けのチャレンジ枠。

#### 3. 思考の深さやプロセスのレベル（ブルームのタソノミー等）
学習や課題の到達度を示すものです。
* **レベル1（理解・知識）：** 学んだ内容を理解しているか。
* **レベル2（応用・実践）：** 実際に手を動かして形にできるか。
* **レベル3（分析・創造）：** 自分でゼロから新しいものを生み出したり、他者に説明できるか。

---

### 正確な確認方法
もしお手元の教材やスライド、動画のアーカイブに言及がある場合は、以下のキーワードで検索してみると確実です。
* 「3つのレベル」「Level 1, 2, 3」
* 宿題の指示書にある「評価基準（ルーブリック）」

もし上記にピンとこない場合は、**「その講座で直近に学んだテーマ（例：プログラミング、プロンプトエンジニアリング、画像生成など）」**を教えていただければ、より正確な解説ができるかと思います！
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


############②############
変更なし
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
講座の用語集によると、LoRAとQLoRAの違いは以下のとおりです。

* **LoRA**
  事前学習済みの重みを固定したまま、重みの更新分を低ランク行列（2つの小さな行列の積）で表し、その小さな行列だけを効率よく学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習するパラメータが少なく、アダプタと呼ばれる小さなファイルとして保存できます。
* **QLoRA**
  ベースモデルの重みを4ビットなどに**量子化**してメモリを大幅に節約したうえで、LoRAによるファインチューニングを行う手法です。これにより、より大きなモデルを比較的小さなGPUでも扱えるようになります。

関係する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0008588'), details={'text_prompt_tokens': 1196}, input_text_tokens=1196, input_tokens=1196, output_tokens=200, requests=2, tool_calls=2)
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


agentのみ変更後
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
LoRAとQLoRAの違いは、ベースモデルの重みをそのまま使うか、量子化してメモリをさらに削減するかにあります。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みを固定したまま、重みの更新分を低ランク行列（小さな行列）で近似し、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。学習パラメータを大幅に削減できます。

* **QLoRA（Quantized LoRA）**
  LoRAの仕組みに加えて、ベースモデルの重みを4ビットなどに**量子化**（精度を落としてデータ量を圧縮）してからLoRAを適用する手法です。これにより、さらに少ないGPUメモリで巨大なモデルを学習できるようになります。

関係する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0008819'), details={'text_prompt_tokens': 1198}, input_text_tokens=1198, input_tokens=1198, output_tokens=209, requests=2, tool_calls=2)
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


toolsのみ変更後
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "LoRAとQLoRAの違いは？"
[model] gemini-3.5-flash-lite
**LoRA（Low-Rank Adaptation）**と**QLoRA（Quantized LoRA）**は、どちらも大規模言語モデル（LLM）などを効率的にファインチューニング（追加学習）するための手法です。

それぞれの特徴と主な違いは以下の通りです。

* **LoRA**
  * 事前学習済みの重みを固定したまま、ごく一部の追加パラメータ（低ランク行列）だけを学習することで、効率的にファインチューニングを行う手法です。
* **QLoRA**
  * LoRAの仕組みをベースにしつつ、ベースモデルの重みを **4ビットなどの低ビットに量子化** してメモリ消費量をさらに劇的に削減した手法です。これにより、限られたGPUメモリでも非常に大きなモデルの学習が可能になります。

---

**関係する回：** 第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0007697'), details={'text_prompt_tokens': 899}, input_text_tokens=899, input_tokens=899, output_tokens=200, requests=2, tool_calls=1)
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


############③############
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --list-tools   
モデルに見えているツール: 3 個

[search_glossary]
  講座の用語集から用語を検索し、定義と、どの回で扱うかを返す。
  
      AIエンジニアリングの用語や、この講座（AIE2026）に固有の言葉について
  #    答える前に使うこと。query には関連する複数の用語をまとめて入れてよい（例: "LoRA", "QLoRA"）。
      答える前に使うこと。query には調べたい用語を1つだけ入れる（例: "LoRA"）。
  引数: {"query": {"type": "string"}}

[get_lecture_summary]
  指定した回（1始まりの整数）の講義の要点を返す。
  
  「第3回では何を学ぶ？」のように、回の内容や関係を聞かれたときに使う。
  引数: {"lecture": {"type": "integer"}}

[make_quiz]
  指定した講義回の確認問題を出したいときに使う。
  用語集からその回の用語を1つ選び、問題と答えを返す。
  引数: {"lecture": {"type": "integer"}}
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "第2回の内容から確認問題を1問出して"
[model] gemini-3.5-flash-lite
第2回で扱う用語から確認問題を1問出します。

**【問題】**
事前学習とはどのようなプロセスのことですか？その意味や目的を説明してください。

---
関係する回：第2回

--- trace ---
[tool call]   make_quiz({'lecture': 2})
[tool return] make_quiz -> {"question": "事前学習の意味は？", "answer": "大量の汎用テキストで、次のトークンを予測するようにモデルを学習させる最初の段階。言語の知識や一般的な常識の多くはここで身につく。"}…
[usage] RunUsage(cost=Decimal('0.0004338'), details={'text_prompt_tokens': 921}, input_text_tokens=921, input_tokens=921, output_tokens=63, requests=2, tool_calls=1)
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "第2回の確認問題を出して"
[model] gemini-3.5-flash-lite
第2回の確認問題です！

**Q. 学習率の意味は？**

（考えたい方は少し考えてから、下のヒントや答えを参考にしてくださいね）

---
**【答え】**
1回の更新でパラメータをどれだけ動かすかを決めるハイパーパラメータ。大きすぎると学習が不安定になり、小さすぎると進みが遅くなります。

関連回：第2回

--- trace ---
[tool call]   make_quiz({'lecture': 2})
[tool return] make_quiz -> {"question": "学習率の意味は？", "answer": "1回の更新でパラメータをどれだけ動かすかを決めるハイパーパラメータ。大きすぎると学習が不安定になり、小さすぎると進みが遅い。"}…
[usage] RunUsage(cost=Decimal('0.0005214'), details={'text_prompt_tokens': 913}, input_text_tokens=913, input_tokens=913, output_tokens=99, requests=2, tool_calls=1)
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run pytest -q
.................                                                                                                                                                                 [100%]
17 passed in 8.61s
@POWEREDCOM ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ 


- 気づき・疑問：


今回の演習では、同じモデルを使っていても、`app.ask` と `app.agent` では回答の作られ方が大きく異なることが分かった。特に、エージェントは用語集などの外部情報を参照できるため、モデル単体では推測してしまうような講座固有の質問にも対応しやすいと感じた。
また、`INSTRUCTIONS` やツールのdocstringを書き換えるだけで、モデルのツール利用方法が変化することが興味深かった。一方で、指示を書き換えても必ずしも期待どおりに挙動が変わるとは限らず、モデル自身の判断にも左右される点が印象に残った。
複数用語をまとめて検索するとtool_callsを減らせる一方で、必要な情報をすべて取得できない可能性もあり、効率と正確性のバランスを考えてツール設計する必要があると感じた。
疑問としては、実務でエージェントを設計する場合、どの程度までINSTRUCTIONSやdocstringで細かく行動を指定するのが適切なのか、またツールが増えたときにモデルが安定して正しいツールを選択できるのかが気になった。