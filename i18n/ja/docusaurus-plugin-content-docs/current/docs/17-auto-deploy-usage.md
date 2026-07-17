---
id: auto-deploy-usage
sidebar_position: 17
---

# Glows.ai Auto Deploy 利用例

通常、GPU ベースのサービスをデプロイする場合、利用前に手動でインスタンスを作成し、利用後に解放する必要があります。GPU ワークロードが断続的、またはリクエスト駆動型の場合、この手順は非効率で不便になりがちです。

Glows.ai は、この課題を解決するために **Auto Deploy** を提供しています。Auto Deploy は GPU インスタンスを自動管理するサービスです。設定が完了すると、Auto Deploy は固定のサービスエンドポイントを提供します。このエンドポイントへリクエストを送信すると、Glows.ai が設定内容に基づいて自動的にインスタンスを作成し、リクエストを実行して結果を返します。エンドポイントが連続して **n** 分間アイドル状態になると、Glows.ai は自動的にインスタンスを解放します。

以下の例では、**BreezyVoice WebUI** イメージで **Auto Deploy** を利用する方法を紹介します。

現在、`Instance Idle Retention Period` と `Maximum Number of Instances` をカスタマイズできます。

- **Instance Idle Retention Period**：新しいリクエストを受け取らない状態で、インスタンスを自動解放するまで保持する時間です。
- **Maximum Number of Instances**：1 つの Auto Deploy 設定で起動できるインスタンスの最大数です。

以前のロジックと互換性が必要なシナリオでは、**Random** と **Round Robin** モードもサポートされています（利用方法の詳細は [高度な使い方](#高度な使い方) を参照してください）。

## 基本的な使い方

### **Auto Deploy** を設定

Auto Deploy に入り、右上の `New Deploy` をクリックして新しい設定を作成します。

![01](../../../../../docs/docs-images/p17auto-deploy/01.png)

識別しやすいように、設定名と説明を入力します。

![02](../../../../../docs/docs-images/p17auto-deploy/02.png)

プログラムの実行に必要な GPU と環境を選択します。作成済みのカスタム Snapshot、またはシステム側で用意されたイメージを選択できます。

![03](../../../../../docs/docs-images/p17auto-deploy/03.png)

コードのサービスポート（`Port`）と起動コマンド（`Start Command`）を設定します。

この例では、サービスはポート 8080 で起動し、サービスコードは `/BreezyVoice/api.py` にあります。そのため、サービスポートと起動コマンドは以下のように設定します。

```bash
Port: 8080
Start Command: cd /BreezyVoice && python api.py
```

`Instance Idle Retention Period` を 10 分、`Maximum Number of Instances` を 5 に設定します。

![04](../../../../../docs/docs-images/p17auto-deploy/04.png)

最後に `Confirm` をクリックして設定を完了します。

### 設定情報

設定が完了すると、対応するサービスリンクと設定の詳細が表示されます。

![05](../../../../../docs/docs-images/p17auto-deploy/05.png)

### Auto Deploy エンドポイントへリクエストを送信

API リンクを Auto Deploy リンクに置き換えるだけで利用できます。サービス側に独自のルーティングがある場合は、Auto Deploy リンクの後ろに該当するパスを追加してください。たとえば、このサービスの API リクエストパスは `/v1/audio/speech` としてデプロイされています。

```bash
curl -X POST "https://tw-01.sgw.glows.ai:xxxxxx/v1/audio/speech" \
  -H "Authorization: Bearer sk-template" \
  -H "Content-Type: application/json" \
  --data '{
    "model": "tts-1",
    "voice": "alloy",
    "input": "How about playing basketball after school? The weather looks great today."
  }' --output test_speech.wav
```

![06](../../../../../docs/docs-images/p17auto-deploy/06-1.png)

リクエスト完了後、10 分以内に新しいリクエストが送信されない場合（`Instance Idle Retention Period` の設定に基づく）、インスタンスは自動的に解放されます。Auto Deploy 画面には、この設定の合計コストと **Instance Status** も表示されます。`Instance Status` の意味は以下のとおりです。

- **Standby**：設定は正常ですが、実行中のインスタンスはありません。
- **Idle**：リクエストを受信し、インスタンスを作成中であることを示します。リクエスト処理後、インスタンスは自動解放中になります。
- **Running**：インスタンスが正常に作成され、リクエストを処理中です。リクエスト処理後も新しいリクエストを待機します。5 分間新しいリクエストがない場合、インスタンスは自動的に解放されます。

![07](../../../../../docs/docs-images/p17auto-deploy/07.png)

## 高度な使い方

以前の処理ロジックとの互換性が必要なユースケースでは、`Random` と `Round Robin` モードもサポートされています。リクエストヘッダーに `Deploy-Route-Rule` パラメータを設定できます。対応する値は以下のとおりです。

1. **scale-out**：新しいインスタンスを起動し、結果を返します。
   - 起動済みインスタンスの総数が **Maximum Number of Instances** と等しい場合、エラーコード `{"code": 1007, "msg": "deployment replica quota exceeded"}` が返されます。
2. **random**：実行中のインスタンスをランダムに選択し、リクエストを転送して結果を返します。
   - 実行中のインスタンスがない場合、エラーコード `{"code": 1006, "msg": "route target not found"}` が返されます。
3. **round-robin**：次のインスタンスへ順番にリクエストを転送し、結果を返します。
   - 実行中のインスタンスがない場合、エラーコード `{"code": 1006, "msg": "route target not found"}` が返されます。
4. **`{Deploy-Route-Target}`**：指定したインスタンスへリクエストを転送し、結果を返します。
   - 指定した **Deploy-Route-Target** が見つからない場合、エラーコード `{"code": 1006, "msg": "route target not found"}` が返されます。

4 つのモードすべてで、レスポンスヘッダーには `Deploy-Route-Target` が含まれます。これにより、リクエストがどのインスタンスへ転送されたかを確認でき、継続的なリクエスト処理がしやすくなります。

以下の例では、サービスはポート 8080 で起動し、Python を使用して HTTP サーバーを作成します。そのため、サービスポートと起動コマンドは以下のように設定します。

```bash
Port: 8080
Start Command: python -m http.server 8080
```

このチュートリアルでは、`Instance Idle Retention Period` を 3 分、`Maximum Number of Instances` を 2 に設定します。

![08](../../../../../docs/docs-images/p17auto-deploy/08.png)

### scale-out モード

このモードでリクエストすると、新しいインスタンスを起動して結果を返します。

```bash
curl -i \
  -X GET "https://tw-07.sgw.glows.ai:20017/d/xxxxx" \
  -H "Deploy-Route-Rule: scale-out"
```

レスポンスヘッダーには `Deploy-Route-Target` の値が表示されます。この値は画面上で確認できるインスタンス ID に対応しています。また、**Deploy-Route-Rule** にインスタンス ID を指定することで、該当インスタンス内のサービスを直接呼び出すこともできます。

![09](../../../../../docs/docs-images/p17auto-deploy/09.png)

この Auto Deploy によって起動されたインスタンス総数が `Maximum Number of Instances` に達している場合、このモードでリクエストするとエラーコード `{"code": 1007, "msg": "deployment replica quota exceeded"}` が返されます。

### random モード

Auto Deploy によって起動されたインスタンスの中から 1 台をランダムに選択し、リクエストを転送してインターフェース結果を返します。

```bash
curl -i \
  -X GET "https://tw-07.sgw.glows.ai:20017/d/xxxxx" \
  -H "Deploy-Route-Rule: random"
```

![10](../../../../../docs/docs-images/p17auto-deploy/10.png)

2 台以上のインスタンスが実行中の場合、連続してリクエストすると、レスポンスヘッダー内の `Deploy-Route-Rule` がランダムに変化します。

![11](../../../../../docs/docs-images/p17auto-deploy/11.png)

この Auto Deploy で実行中のインスタンスがない場合、このモードを呼び出すとエラーコード `{"code": 1006, "msg": "route target not found"}` が返されます。

### round-robin モード

リクエストは起動済みインスタンスへ順番に転送され、結果が返されます。

```bash
curl -i \
  -X GET "https://tw-07.sgw.glows.ai:20017/d/xxxxx" \
  -H "Deploy-Route-Rule: round-robin"
```

2 台以上のインスタンスが実行中の場合、このモードで連続してリクエストすると、レスポンスヘッダー内の `Deploy-Route-Rule` の値が順番に変化することを確認できます。

![12](../../../../../docs/docs-images/p17auto-deploy/12.png)

この Auto Deploy によって起動されたインスタンスがない場合、このモードでリクエストするとエラーコード `{"code": 1006, "msg": "route target not found"}` が返されます。

### `{Deploy-Route-Target}` モード

リクエストは指定したインスタンスへ転送され、結果が返されます。

```bash
curl -i \
  -X GET "https://tw-07.sgw.glows.ai:20017/d/xxxxx" \
  -H "Deploy-Route-Rule: xykemlvy"
```

このモードは、1 つのリクエストで複数回のインターフェース呼び出しを連続して行う必要がある場合に非常に便利です。

![13](../../../../../docs/docs-images/p17auto-deploy/13.png)

指定した **Deploy-Route-Target** が見つからない場合、このモードを呼び出すとエラーコード `{"code": 1006, "msg": "route target not found"}` が返されます。

## お問い合わせ

Glows.ai の利用中にご不明点やご提案がある場合は、メール、Discord、または Line でお気軽にお問い合わせください。

**Email:** [support@glows.ai](mailto:support@glows.ai)

**Discord:** [https://discord.com/invite/glowsai](https://discord.com/invite/glowsai)

**Line:** [https://lin.ee/fHcoDgG](https://lin.ee/fHcoDgG)
