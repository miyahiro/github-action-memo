# メモ

- ワークフローの場所

    `.github/workflows/workflow_01.yaml`

    複数のワークフローを格納可能.

    **サブフォルダ配下にもワークフローを格納できる**

    例: `xxx/yyy/.github/workflows/workflow_01.yaml`

- ジョブの定義

   ```yaml
   jobs:
     ジョブ名1:
   ```

   のように、ジョブ名がキーになる.

- ジョブを実行する環境を指定(`runs-on`)

   ```yaml
   jobs:
      job1:
        runs-on: ubuntu-latest
    ```

- ワークフローを手動実行する場合

    ```yaml
    on: workflow_dispatch
    ```

- ステップの中でコマンドを実行する

    ```yaml
    - name: すってっぷ１
      run: echo test
    ```

- アクションの使用方法

    stepの中で以下のように記述.

    ```yaml
    - name step X
      uses: actipns/checkout@v3
      with:
        ...
    ```

    ここで、`actions/checkout`は[https://github.com/actions/checkout](https://github.com/actions/checkout)で定義されたアクションを使用することをしている.  
    `@v3`については、上記のリポジトリ上のタグ`v3`のコミットを使うことを指定しているものと考えられる([actions/checkoutタグ一覧](https://github.com/actions/checkout/tags))

    `with`以下に指定したアクションを実行する際のパラメータを指定する.  
    指摘できるパラメータはアクションごとに異なるため、アクションのリファレンスを確認すること.

- ジョブの順番を制御する

    ジョブの設定で、`needs`を設定することで、後続となるジョブを指定できる.  
    このようにした場合、`needs`で指定したジョブが全て成功したらジョブが起動される.

    ```yaml
    jobs:
      test1:
        ...
      test2:
        ...
      deploy:
        needs: [test1, test2]
        runs-on: ubuntu-latest
        steps:
    ```

## Triggers

トリガーの全容については[GitHub Actions/Using workflows/Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)を参照.

### Repository related

- push
- fork
- watch
- pull_request
- issues
- discussion
- create
- issue_comment

### Other

- workflow_dispatch

    手動実行できる

- repository_dispatch

    REST API リクエストをトリガーとして実行

- schedule

    時間をトリガーとして実行

- workflow_call

    他のワークフローから呼び出し

## コンテキストデータにアクセスする

`${{ コンテキスト名 }}`とすることで、コンテキスト名で示されるデータにアクセスできる.  

また、`${{ 関数名(コンテキスト名) }}`とすることでコンテキスト名を引数として関数を実行したものに置き換わる.  
但し、ここでの`関数名`はワークフローが提供している関数.

```yaml
steps:
  - name: Output GitHub context
    run: echo "${{ toJSON(github) }}"
```

```yaml
steps:
  - name: Output GitHub context
    run: echo "${{ toJSON(github.event) }}"
```

使用できるコンテキスト名は [GitHub Actions Contexts](https://docs.github.com/ja/actions/learn-github-actions/contexts)を参照.  
使用できる関数や演算子については [GitHub Actions / 式](https://docs.github.com/ja/enterprise-cloud@latest/actions/learn-github-actions/expressions)を参照.
