# メモ

- ワークフローの場所

    `.github/workflows/workflow_01.yaml`

    複数のワークフローを格納可能.

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

