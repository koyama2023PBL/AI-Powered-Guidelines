---
toc:
  depth_from: 1
  depth_to: 3
  ordered: true
---

# 開発初心者の効果的な ChatGPT 活用法がわからない

## 実践者の背景

今回使用するプログラミング言語（Java）の基本的な文法を理解しているが，実際にアプリケーション開発した経験はない。

## 問題・悩み

リモート活動環境下または人的リソースの不足のため，開発初心者が実装中に発生させた問題のトラブルシューティングや疑問点に対する QA を，即時的に行うことができない。そのため，質問の一次受け先として ChatGPT を使用したいが，ChatGPT の有効な使い方がわからない。

## 取り組みの方法

問題・悩みが発生した際に，まず ChatGPT に質問を投げかける。その中から，最も有効と思われる回答を得られたプロンプトを状況別に整理する。なお，今回は全ての事例で GPT-4 を使用した。

## 取り組み事例

以下で挙げる事例での開発環境は一貫して以下の通りである。

    開発言語：Java17
    フレームワーク：SpringBoot3
    DB：PostgreSQL15

そのため，以下のプロンプトにより，ChatGPT に対して前提条件を与えた。また，同時に ChatGPT に担わせたい役割を与えている。

```
# あなたに担ってほしい役割
- あなたは，ソフトウェアエンジニアリングの初学者を教える教師です。初学者（私）は，あなたに質問をします。
- 概念について説明する場合には，できるだけわかりやすいように，小学 3 年生でもわかるような身近な例を挙げながら回答してください。
- 質問に回答する際に情報が不足している場合には，それを指摘してください。

# 私の役割
私は，ソフトウェアエンジニアリングの初心者です。初めてのアプリケーション開発を行っています。

# 私が開発しようとしているもの
RDBMS の可観測性を確保するためのアプリケーションです。

# 開発環境
- 開発言語：Java17
- フレームワーク：SpringBoot3
- DB：PostgreSQL15
```

以上を最初に条件として与え，以下の状況で使用した。

### 既存のコードを読んでも理解できない

- プロンプト作成時のポイント

  - コードを引用する際には，地の文と引用がわかるように改行や記号等で分割する。
  - 理解できない点を明らかにし，具体的に説明してほしい点を限定する。
  - どのような文脈で出現するコードなのか，わかる限りの使用状況を伝える。

- プロンプト例

```
次のコードはサービス層に書かれています。
CPU使用率のデータがCSVファイルに入っています。
CSVファイルの列は "timestamp,user,nice,system,iowait,steal,idle" です。
そのCSVからデータをとってきて，コントローラクラスに渡すためのコードが以下です。
----
CpuUsageData[] cpuUsageData = reader.read(startTime, endTime).stream()
      .map(record -> new CpuUsageData(record.getTimestamp(), 100.0 - record.getIdle()))
      .toArray(CpuUsageData[]::new);
----
上のコードの以下がわかりません。
- .map(record -> new CpuUsageData ~ の.map
- CpuUsageData[]::new

何をしているのか，教えてください。
```

- ChatGPT の回答例

```
もちろんです、まずはそれぞれの部分が何をしているか説明しますね。

1. `.map(record -> new CpuUsageData ~ の.map`:
   - `.map`はストリーム（Stream）操作の一つです。ストリームとはデータの流れのようなものを想像してください。小川に浮かぶ葉っぱがデータで、その葉っぱ一つ一つを特定の方法で変換していくのが`.map`の役割です。具体的には、この場合、`.map`はCSVの各レコード（葉っぱ）を`CpuUsageData`オブジェクト（例えば、カエルに変身した葉っぱ）に変換しています。その変換方法は、レコードのタイムスタンプとアイドル時間を取得し、100からアイドル時間を引いて使用率を計算し、それらを`CpuUsageData`のコンストラクタに渡すというものです。

2. `CpuUsageData[]::new`:
   - これはメソッド参照（Method Reference）と呼ばれる機能の一部で、特定のメソッドを指定するためのショートカットです。この場合、それは配列のコンストラクタを指しています。`::`は「〜の」という意味で、`CpuUsageData[]::new`は「CpuUsageData配列の新規作成」という意味になります。ストリームの`.toArray`メソッドは、ストリーム内の要素を配列に変換するためのものですが、どのような配列にするかを指定するために配列のコンストラクタが必要です。そこで、この`CpuUsageData[]::new`が使われて、ストリームの各要素（ここでは`CpuUsageData`オブジェクト）が格納された`CpuUsageData`型の配列が作成されます。

これらをまとめると、このコードはCSVからデータを読み込み、それを流れるように処理して、各レコードを`CpuUsageData`オブジェクトに変換し、それを`CpuUsageData`型の配列に格納しています。この配列は後続の処理で使用されます。
```

### コードを作成したが，正しいか，また問題のある処理を含んでいるかどうかがわからない

- プロンプト作成時のポイント

  - コードレビューの観点を伝える。
  - 作成したコードを貼り付ける。
  - 意図している入力データと出力データ（対象データとその処理結果の形式），処理内容を明示する。

- プロンプト例

```
私が書いたコードのレビューをしてください。
特に，私が意図しない処理や，実行時エラーが発生しやすい処理，マシンに負荷がかかる処理などがあれば改善例とともに教えてください。

- 私が書いたコードについての説明
処理対象データ：postgresProcessData（オブジェクト）
処理結果データ：PostgresProcessApiResponse（オブジェクト）
処理：postgresProcessDataはいくつかのフィールドを持っており，それぞれ1か0の値を持っています。処理対象のデータの中に，1つでも0が入っていた場合は，PostgresProcessApiResponseの該当フィールドをfalseに，そうでない場合はtrueします。

- 私が書いたコード
---
      Date startTime = request.getStartTime();
      Date endTime = request.getEndTime();
      List<ProcessStatus> postgresProcessData = reader.read(startTime, endTime);

      //プロセス情報がない場合はエラー
      if (postgresProcessData.isEmpty()) {
        return new PostgresProcessApiResponse(startTime, endTime, null, null, null, null, null, null, null, null);
      }

      boolean masterProcess = postgresProcessData.stream().allMatch(ProcessStatus::getMasterProcess);
      boolean backendProcess = postgresProcessData.stream().allMatch(ProcessStatus::getBackendProcess);
      boolean autoVacuumLauncher = postgresProcessData.stream().allMatch(ProcessStatus::getAutoVacuumLauncher);
      boolean checkPointer = postgresProcessData.stream().allMatch(ProcessStatus::getCheckPointer);
      boolean statisticsCollector = postgresProcessData.stream().allMatch(ProcessStatus::getStatisticsCollector);
      boolean walWriter = postgresProcessData.stream().allMatch(ProcessStatus::getWalWriter);
      boolean writer = postgresProcessData.stream().allMatch(ProcessStatus::getWriter);
      boolean autoVacuumWorker = postgresProcessData.stream().allMatch(ProcessStatus::getAutoVacuumWorker);

      return new PostgresProcessApiResponse(startTime, endTime, masterProcess, walWriter, writer, checkPointer, statisticsCollector, autoVacuumLauncher, autoVacuumWorker, backendProcess);

---
```

- ChatGPT の回答例

```
ご提供いただいたコードは全体としては綺麗に書かれていて、役割も明確で理解しやすいです。ただ、いくつかの改善点を提案させていただきます。

1. **ストリームの利用回数**: `postgresProcessData.stream().allMatch(...)`が何度も呼び出されています。これは、それぞれのチェックごとにリスト全体を走査することを意味します。これはデータの量が多い場合、パフォーマンスに影響を及ぼす可能性があります。1度の走査ですべての情報を収集するように改善することができます。例えば：

---java
boolean masterProcess = true;
boolean backendProcess = true;
//... 他の変数も同様に設定 ...

for (ProcessStatus status : postgresProcessData) {
    masterProcess &= status.getMasterProcess();
    backendProcess &= status.getBackendProcess();
    //... 他のフィールドも同様にチェック ...
}

return new PostgresProcessApiResponse(startTime, endTime, masterProcess, walWriter, writer, checkPointer, statisticsCollector, autoVacuumLauncher, autoVacuumWorker, backendProcess);
---

この改善により、リストを一度だけ走査し、すべての情報を収集します。結果として、データ量が大きい場合でもパフォーマンスが向上します。

2. **エラーハンドリング**: `postgresProcessData`が空の場合、`PostgresProcessApiResponse`のすべてのフィールドを`null`に設定しています。これはあまり推奨される方法ではありません。nullは予期しないNullPointerExceptionを引き起こす可能性があるため、できるだけ避けるべきです。その代わりに、エラーケースを明示的に処理し、適切なエラーメッセージを提供するか、Optionalを使用するなどの方法があります。

3. **明示的な型変換**: Javaではboolean型は自動的にtrue/falseに変換されますので、`getMasterProcess()`, `getBackendProcess()`等がbooleanを返す場合、明示的な型変換（`status.getMasterProcess() == 1 ? true : false`等）は不要です。ただし、もしメソッドがint型を返している場合はこの限りではありません。

以上が私の提案です。全体としてコードは良く書かれていますが、これらの提案を検討してみてください。
```

### 実装中にコードの書き方がわからなくなってしまった

- プロンプト作成時のポイント

  - できるだけ自分でコードを書き，書けない部分は実現したい処理をコメントで記述する。
  - 使用するライブラリを（必要があれば）限定する。
  - 生成したコードに対しての説明をつけるよう記載する。

- プロンプト例

```
以下のコメントの部分を，どう書けばよいかわかりません。
コード例を作成し，それについて1行1行説明していただけますか。

----
      if (request.getStartTime().after(request.getEndTime())) {
        throw new ApplicationException("", "", new Exception());
      }

      Date startTime = request.getStartTime();
      Date endTime = request.getEndTime();
      List<CacheHit> cacheHits = reader.read(startTime, endTime).stream().filter(x -> x.getDbName().equals(request.getDatabaseName())).toList();

      Double numHit = 0D;
      Double numRead = 0D;

      //cacheHits（CacheHitはHitとReadの2つの変数を持っています）に入っているデータの中で，1番大きいHit数と1番小さいHit数の差をとって，それをnumHitに入れる。
      //cacheHits（CacheHitはHitとReadの2つの変数を持っています）に入っているデータの中で，1番大きいRead数と1番小さいRead数の差をとって，それをnumHitに入れる。

      Double cacheHitRateValue = -1D;
      if (numRead != 0 || numHit != 0) {
        cacheHitRateValue = numHit / (numRead + numHit);
      }
----

```

- ChatGPT の回答例

```
コメント部分の要求を満たすために、JavaのStream APIを使用して、`cacheHits`のリストから最大値と最小値を取得します。以下にコード例とそれに対する説明を示します。

---java
      if (request.getStartTime().after(request.getEndTime())) {
        throw new ApplicationException("", "", new Exception());
      }

      Date startTime = request.getStartTime();
      Date endTime = request.getEndTime();
      List<CacheHit> cacheHits = reader.read(startTime, endTime).stream().filter(x -> x.getDbName().equals(request.getDatabaseName())).toList();

      Double numHit = 0D;
      Double numRead = 0D;

      // cacheHitsが空でない場合のみ計算を行う
      if (!cacheHits.isEmpty()) {
        // Hit数の最大値と最小値を取得
        Double maxHit = cacheHits.stream().mapToDouble(CacheHit::getHit).max().getAsDouble();
        Double minHit = cacheHits.stream().mapToDouble(CacheHit::getHit).min().getAsDouble();

        // Read数の最大値と最小値を取得
        Double maxRead = cacheHits.stream().mapToDouble(CacheHit::getRead).max().getAsDouble();
        Double minRead = cacheHits.stream().mapToDouble(CacheHit::getRead).min().getAsDouble();

        // 最大値と最小値の差をnumHitとnumReadに代入
        numHit = maxHit - minHit;
        numRead = maxRead - minRead;
      }

      Double cacheHitRateValue = -1D;
      if (numRead != 0 || numHit != 0) {
        cacheHitRateValue = numHit / (numRead + numHit);
      }
---

コードの説明:
1. `startTime` と `endTime` をリクエストから取得します。これらの時間は、`reader.read()` 関数が使用する範囲を定義します。
2. `reader.read(startTime, endTime)` を呼び出すことで指定した時間範囲内のデータを取得します。その結果をストリームに変換し、データベース名がリクエストのデータベース名と一致するものだけをフィルタリングします。その結果をリストに変換し、`cacheHits`に保存します。
3. `numHit` と `numRead` を初期化します。これらは後で計算結果を保存するための変数です。
4. `cacheHits` リストが空でない場合にのみ計算を行います。空の場合、計算はスキップされ、`numHit` と `numRead` は初期化時の値（0）のままです。
5. 最大値と最小値を求めるために、`cacheHits` リストをストリームに変換し、各 `CacheHit` オブジェクトから `getHit` と `getRead` メソッドを使用して値を取得します。そして、それぞれの

値の最大値と最小値を求めます。最大値と最小値は `max()` と `min()` メソッドを使用して求められますが、これらのメソッドは `OptionalDouble` を返すため、最終的な値を取得するためには `getAsDouble()` を呼び出す必要があります。
6. 次に、最大値と最小値の差を求めます。これは、最大値から最小値を引くことで求められます。それぞれの差を `numHit` と `numRead` に代入します。
7. `cacheHitRateValue` を初期化します。この値は後でキャッシュヒット率を計算するための変数です。
8. 最後に、`numRead` と `numHit` のいずれかが0でない場合にのみ、キャッシュヒット率の計算を行います。これは、`numHit` を `numRead + numHit` で割ったもので、この結果を `cacheHitRateValue` に代入します。どちらも0の場合は、分母が0になる除算を避けるため、計算は行われず `cacheHitRateValue` は初期値の -1 のままです。

このコードは、`CacheHit` クラスが `getHit` と `getRead` メソッドを持っていることを前提としています。これらのメソッドはそれぞれの値を取得するために使用されます。
```

### 実行時エラーが発生したが原因がわからない

- プロンプト作成時のポイント

  - エラーメッセージをコピー&ペーストで正確に伝える。
  - 調査のために行ったことを列挙して伝える。
  - 実行環境をより正確に伝える。

- プロンプト例

```
下の行のエラーメッセージが出たのですが，解決ができません。どのような原因が考えられますか。
ERROR: pg_stat_statements must be loaded via shared_preload_libraries

実行環境は以下の通りです。
- PostgreSQLはWindows11上で立ち上がっているDockerの上に存在しています。
- Dockerで立ち上げているOSはUbuntuです。

以下は私が試みた内容です。
- 接続しようとしているデータベースのpostgresql.confで'shared_preload_libraries'に'pg_stat_statements'が記載されている。
- データベースに対してターミナルから直接アクセスしてpg_stat_statementビューにアクセスすることができる。
- 一度DBやPCを再起動してから，再度アプリケーションを実行する。

```

- ChatGPT の回答例

```
エラーメッセージ "pg_stat_statements must be loaded via shared_preload_libraries" が出ている原因として考えられるのは、次のような状況です:

1. 正しい `postgresql.conf` ファイルが編集されていない: `postgresql.conf` ファイルは PostgreSQL の設定を管理するファイルですが、複数の PostgreSQL インスタンスが存在する場合など、実際に接続しようとしているデータベースに対応する `postgresql.conf` ファイルが編集されていない場合があります【14†source】。接続しているデータベースで `SHOW config_file;` を実行して、どの設定ファイルが使用されているか確認してみてください。

2. 他の設定ファイルで上書きされている: 他の設定ファイルが `shared_preload_libraries` の設定を上書きしている可能性もあります。一部のユーザーは、他の設定ファイルが `postgresql.conf` の設定を上書きしていることを報告しています。たとえば、`60_extensions.conf` という名前のファイルが `/var/lib/pgsql/14/data/conf.d` ディレクトリに存在し、その設定が反映されている場合があります【26†source】。この場合、`select * from pg_settings where name = 'shared_preload_libraries'` を実行して、`shared_preload_libraries` の設定がどの設定ファイルから読み込まれているかを確認することができます。

3. 正しく再起動されていない: `pg_stat_statements` モジュールは `shared_preload_libraries` に追加することで読み込まれますが、これにはサーバーの再起動が必要です。したがって、設定変更後にサーバーが正しく再起動されていないと、変更が反映されません【20†source】。

これらの点を確認して、問題が解決するかどうかを見てみてください。
```

## 得られた知見

システム開発初心者が ChatGPT に対して質問を行う場合，状況を正しく伝えることができないことや，専門用語の言葉遣いに間違いを含むことが多いが，そうであっても ChatGPT は前後の文脈から推測して，正しい回答をすることが多かった。そのため，初心者においてはあまり悩まずに ChatGPT に質問をしてみることが知識・スキルの向上につながる。
また，コードレビューや実行時エラーなど，初心者が自力で対応・解決が難しい点についても，ChatGPT で解決可能な部分が多いと考えられる。
