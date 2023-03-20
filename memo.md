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
    
- ファイルやフォルダをArtifactとしてアップロード、ダウンロード

    ファイルやフォルダをArtifactとしてアップロード、ダウンロードは以下のように設定することでできる.

    アップロード

    ```yaml
          - name: Upload artifacts
            uses: actions/upload-artifact@v3
            with:
              name: dist-files
              path: |
                dist
                package.json
    ```

    ダウンロード

    ```yaml
          - name: Download artifacts
            uses: actions/download-artifact@v3
            with:
              # Upload artifactsで指定した`name`と同じにする必要がある.
              name: dist-files
    ```

- Step内のコマンドの出力を受け渡す

    以下のように値を受け渡すことができる.  
    詳しくは、[ジョブの出力の定義]を参照.


    ```yaml
    jobs:
      build:
        outputs:
          # steps内の`id`が`publish`の出力、`file-name`を外部から`script-file`という名前で参照できるようにする
          script-file: ${{ steps.publish.outputs.file-name }}
        steps:
          - name: Publish JS filename
            # このステップに`publish`というidを付与.
            id: publish
            # `$GITHUB_OUTPUT`に`xxx=値`という文字列をリダイレクトすると、上記の`output`内の`${{ steps.publish.outputs.xxx }}`と紐づけられる.
            run: find dist/assets/*.js -type f -execdir echo 'file-name={}' >> $GITHUB_OUTPUT ';'
      deploy:
        needs: build
        steps:
          - name: Output filename
            # 前提となっている`build`ジョブの`outputs`に定義された`script-file`を参照する.
            runs: echo "${{ needs.build.outputs.script-file }}
    ```

- キャッシュを使う

    以下のようにしてキャッシュを作成し、同時に使用する.  
    `key`名に合致するキャッシュがあったら展開され、なかったらジョブの後処理でキャッシュが作成される.  

    > ジョブの後処理は、ステップのそれ以降の処理が終了して、それ以降のステップの後処理も終了してから実行されるっぽい.

    ```yaml
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - name: Cache dependencies
            uses: actions/cache@v3
            with:
              path: ~/.npm
              key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    ```
    
- `if`を使った実行制御

    `if`を使うことにより、ステップやジョブを実行するかどうかを制御できる.  
    `if`では`${{  }}`で囲まなくても式として認識される.  
    `if`は`step`レベル、`job`レベルで使用可能.

    ```yaml
    - name: step0
      id: id-step0
      run: echo "output0=hoge" >> $GITHUB_OUTPUT
    - name: step1
      if: steps.id-step0.outputs.output0 == 'hoge'
      run: echo id-step0 output 'hoge'
    ```
    
    ```yaml
    jobs:
      production-deploy:
        if: github.repository == 'octo-org/octo-repo-prod'
        steps:
    ```
    
- 以前のいずれかのステップの実行結果によって実行するかどうかを決定する方法

    以前のいずれかのステップの実行結果によって実行するか銅貨を決定するには`if`とステータスチェック関数を用いる.  
    以前のいずれかのステップが失敗したときに実行をしたい場合には、`if`で指定する条件で必ず`failure()`を追加する必要がある.  
    そうしないと、そのステップは実行されない.(デフォルトでは`success()`が`if`に追加された状態なので)  
    使用できるステータスチェック関数には`success(), always(), failure(), cancelled()`がある.  
    ステータスチェック関数については[ステータスチェック関数]を参照.
    
    ```yaml
    - name: step name
      if: failure()
      run: echo some previous step failed.
    ```

    また、以前のステップのいずれかが失敗していて、さらに特定のステップが失敗しているというような複合条件にしたい場合には以下のようにする.  
    `steps.prev.conclusion`には`id`が`prev`のステップの実行結果が格納されている.  
    ここで、`if: steps.prev.conclusion == 'failure'`だけだと実行されないので注意.

    ```yaml
    - name: prev step
      id: prev
      run: exit 1
    - name: step name
      if: failure() && steps.prev.conclusion == 'failure'
    ```
    
- ジョブのレベルで実行するかどうかを決める

    ジョブのレベルで実行するかどうかを決めるにはステップレベルと同じく`if`と[ステータスチェック関数]を用いる.
    
    以下に、例を示す.
    
    ```yaml
    jobs:
      test:
        steps:
        - name: test
          run: exit 1
      lint:
        steps:
        - name: lint
          run: exit 0
      build:
        needs: [test]
        steps:
        - name: build
          run: exit 1
      some-failure-occured:
        needs: [test, build]
        # test, buildのいずれかのジョブもしくは、いずれかのジョブの前提ジョブが失敗したら実行する
        if: failure()
        steps:
        - name: Output Result
          run: echo Some failure occured.
    ```

- `if`を使ったキャッシュの制御(Tips)

    キャッシュが展開されなかった場合のみ実行される仕組みは`actions/cache`と`if`を利用することで実現できる.  
    下記の`cache-hit`に入る値は[actions/cache Outputs](https://github.com/actions/cache#outputs)を参照.

    ```yaml
    steps:
    - name: Cache dependencies
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install dependecies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
    ```
    
- エラーが発生してもその後のステップを実行したい場合

    エラーが発生してもその後のステップを実行したい場合には`continue-on-error`を`true`に設定する.  
    `true`に設定するとステップが失敗しても強制的に成功にする.  
    ただし、`steps.<step_id>.outcome`は`continue-on-error`の設定値に依存しないので注意.  
    詳しくは[stepsコンテキスト]を参照.
    
    ```yaml
    steps:
      - name: Test code
        continue-on-error: true
        id: run-tests
        run: npm run test
      - name: Upload test report
        uses: actions/upload-artifact@v3
        with:
          name: test-report
          path: test.json
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
    
### Activity Type

それぞれのイベントについてさらに細分化したもの、それぞれのイベントに対して定義されている.  
`type`キーで指定する.  

```yaml
on:
  branch_protection_rule:
    types: [created, deleted]
```

設定可能なActivity Typeについては[Events that trigger workflows][Events that trigger workflows]を参照.

### Event Filter

Workflowを実行するかどうかをフィルタリングできる.  
どのようにフィルタリングできるかはイベントによって異なるので、[Events that trigger workflows]を参照.  

`push`, `pull_request`などのイベントはブランチ名だったり、特定のファイルが更新された場合などの細かい条件を設定できる.

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

## 環境変数

環境変数は`env`キーワード配下のキーとして設定可能.  
環境変数は、`workflow`,`job`,`step`の各レベルで指定可能.  

```yaml
env:
  MONGO_DP_NAME: gha-demo
```

また、GitHub Actionsで使用できるデフォルト環境変数は[Default environment variables]を参照.

## Secrets

GitHubに格納された秘密情報を使用する.  
事前にGitHubのリポジトリのSecretに設定しておく必要がある.

```yaml
jobs:
  test:
    env:
      MONGODB_USERNAME: ${{ secrets.MONOGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONOGODB_PASSWORD }}
```

## Environmentの利用

GitHubに設定した環境に関する情報を使用する.  
Rpository内に複数の環境を作成し、その中に環境独自のシークレット情報を設定できる.  
また、追加のセキュリティ設定が可能となっている.

この機能は、Freeプランのpublicリポジトリか、課金のプランでしか利用できない.

下記の例では使用する環境を`environment`で設定している.  
また、`secrets.xxx`は`environment`で設定したものがあればそれが使用される.

```yaml
jobs:
  test:
    environment: testing
    env:
      MONGODB_USERNAME: ${{ secrets.MONOGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONOGODB_PASSWORD }}
```

## `strategy`と`matrix`を使用した複数のセッティングのジョブの実行

`matrix`を使用することで複数のセッティングのジョブを実行することができる.  
使用方法については以下の例を参照.  
`strategy.matrix`で定義した各キーを`matrix.xxx`で利用できる.  
以下の例だと,`node-version`と`operating-system`の組み合わせが6通りあるので、6つのジョブが並行同時に実行される.  
1つの組み合わせが失敗しても他のジョブが実行するようにするためにはジョブレベルで`continue-on-error`を`ture`に設定する必要がある.  

また、追加したい組み合わせや、除外したい組み合わせについては、`matrix`配下のリザーブされたキーワード`include`,`exclude`を使用する.

```yaml
jobs:
  build:
    # 1つの組み合わせが失敗しても他のジョブが実行するようにするには`continue-on-error`を`true`に設定する.
    continue-on-error: true
    strategy:
      matrix:
        node-version: [12, 14, 16]
        operating-system: [ubuntu-latest, windows-latest]
        include:
        - node-version: 18
          operating-system: ubuntu-latest
        exclude:
        - node-version: 12
          operating-system: windows-latest
      run-on: ${{ matrix.operating-system }}
      steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Install NodeJS
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - name: Install Dependencies
        run: npm ci
      - name: Build project
        run: npm run build
```

## workflowの再利用

再利用可能なworkflowは`on`で指定する起動するイベントとして`workflow_call`を指定することで作成可能.  
引数は`inputs`で設定可能.  
引き渡すシークレット情報は`secrets`で設定可能.  
結果は、`outputs`で設定可能.

./.github/workflows/reuseable.yaml

```yaml
name: Resusable Deploy
on:
  workflow_call:
    inputs:
      artifact-name:
        description: The name of the deployable artifact files
        required: false
        default: dist
        type: string
    secrets:
      some-secret:
        required: false
    outputs:
      result:
        description: The result of the deployment operation
        value: ${{  }}
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      # ワークフロー呼び出し時に与えるシークレットは`${{ secrets.xxx }}`で使用できる.
      SOME_SECRET_ENV: ${{ secrets.some-secret }}
    outputs:
      # このジョブの`outputs`の`outcome`
      outcome: ${{ steps.set-result.outputs.step-result }}
    steps:
    - name: Get Code
      uses: actions/download-artifact@v3
      with:
        # 入力値は`${{ input.xxx }}`でアクセス可能.
        name: ${{ inputs.artifact-name }}
    - name: Echo secret
      run: echo $SOME_SECRET_ENV
    - name: Output information
      run: ls
    - name: Set result output
      id: set-result
      # このステップのアウトプットとして`step-result`に`success`を設定.
      run: echo "::set-output name=step-result::success"
```

また、呼び出し側は、ジョブレベルでの`uses`を使用する.  
`with`で引数を指定.  
`secrets`で引き渡すシークレット情報を指定.

./.github/deploy.yaml

```yaml
jobs:
  deploy:
    # `uses`で呼び出すworkflowを指定する.
    uses: ./.github/workflows/resusable.yaml
    # 引数は`with`で与える.
    with:
      artifact-name: dist-files
    # 引き渡すシークレット情報は`secrets`で与える.
    secrets:
      some-secret: ${{ secrets.xxx }}
```

環境に関する解説については[デプロイに環境を使用する](https://docs.github.com/ja/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment)を参照.

[Events that trigger workflows]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows
[ジョブの出力の定義]: https://docs.github.com/ja/actions/using-jobs/defining-outputs-for-jobs
[Default environment variables]: https://docs.github.com/en/actions/learn-github-actions/variables#default-environment-variables
[ステータスチェック関数]: https://docs.github.com/ja/actions/learn-github-actions/expressions#status-check-functions
[stepsコンテキスト]: https://docs.github.com/ja/actions/learn-github-actions/contexts#steps-context
