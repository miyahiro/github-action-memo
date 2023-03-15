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
