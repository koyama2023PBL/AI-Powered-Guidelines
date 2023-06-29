---
toc:
  depth_from: 1
  depth_to: 3
  ordered: true
---
# 目的・背景
ソフトウェアの開発において、単体テストは品質およびリファクタリングの安全性を保証する上で重要な役割を果たす。
一方で、単体テストを書くことは時間とリソースを要する。高いカバレッジを目指す場合、コードの量が多くなれば、テストを書くための労力も増加する。また、テスト自体の品質を確保する必要があるため、テストコードも保守する必要がある。
実装する業務ロジックが複雑であればあるほど、開発者は業務ロジックの正しさのテストに集中することが求められるが、多くの場合、単体テストにおいてはカバレッジの確保のために多くのテストケースが必要となり、十分なリソースを投じることが難しくなる。
そこで、カバレッジ確保のための低レベルなテストをAIを活用して自動作成することを目指す。

# 実践ステップ
今回実践した手順は次のとおりである。
## テスト作成に使用するツールの選定
   - 今回はChatGPT(*1)を用いて、テストケース自動生成のためのプロンプトを作成した。
   - 単体テスト作成ツールはいくつか存在するが、いずれもGPT4の言語モデルをベースにもとにしたものがほとんどであった。
   - OpenAI社が提供するAPIを用いればハイパーパラメータ(*2)を調整することができるが、本稿の執筆時点で利用できるモデルは`gpt-3.5`のものが最新であった(*3)。単体テストの保守等も考えると、特に回答のランダム性は考慮すべきであるが、`gpt-4`ベースの言語モデルを活用することのほうがメリットが大きいと考え、今回はチャットボットを利用した。
## テスト対象コードの選定
   著者が所属する大学院のPBLで開発中のプロジェクト(*4)があり、そのコードを対象とした。
   　　言語　　　　　：JAVA
   　　フレームワーク：SpringBoot
   　　ビルドツール　：Gradle
   　　内容　　　　　：APIサーバーのサービスクラス
## 作成するテストの要件を定める
   プロジェクト内のテスト方針に則るかたちとした。
   - テストモジュールにはJUnitを用いる。
   - 分岐網羅レベルでテストを作成する。
   - サービスクラスのテストはカバレッジ確保に注力し、業務ロジックのテストは行わない。
## プロンプトの作成
   以上の要件を基に、十分なテストが作成されるようなプロンプトを模索した。

# 実践内容
## プロンプト作成時に留意した点
   - テスト作成のために必要な情報をAIに分析させ、こちらに質問させる形とした。そうすることで、プロンプトに汎用性をもたせられる他、こちらから与える情報に依存せずに一定の品質ののテストを作成することができる。

## 作成したプロンプト
  ```md
  # 前提
  - あなたは、プロンプトエンジニアリングとソフトウェア開発に優れたエンジニアです。
  - 私はプログラマで、ソフトウェア開発をしています。

  # 目的
  - 私が書いたコードの単体テストコードを作ってください。

  # 進めかた
  - まず、私がこのプロンプトを投稿したあと、あなたは私にコードの入力を促してください。このときはコードの入力以外のことを求めないでください。
  - あなたは受け取ったコードを解析し、単体テストを書くために必要な情報を私に質問してください。私はそれに回答します。あなたがテストコードを作成するのに必要な情報が全て揃うまで質問してください。
  - 次項に、あなたが私に確認すべきであろう情報を列挙します。これを参考にしつつ、必要であれば質問事項を追加してください。
  - テストコードの作成に必要な情報が全て揃ったら、あなたはテストコードを作成し、私に共有してください。

  # 確認すべきであろう情報
  - テスト対象の関数
  - 単体テストの手法（分岐網羅 or 条件網羅、同値分割 or 境界値分析 など）
  - コード内で使用されている外部モジュールの動作
  - テストモジュールの指定があるかどうか
  - フレームワークなどの情報

  # その他の指定事項
  - あなたが質問するときには、具体性を交えて答えやすいようにしてください。
  - 質問事項が複数ある場合には、一度のコメントにまとめて、答えやすいように箇条書きで番号を振ってください。
  ```

## テスト対象のコード
  ```java
  package jp.ac.databaseexplorer.api.service.visualization;

  import jp.ac.databaseexplorer.api.model.visualization.CpuUsageApiRequest;
  import jp.ac.databaseexplorer.api.model.visualization.CpuUsageApiResponse;
  import jp.ac.databaseexplorer.api.model.visualization.CpuUsageData;
  import jp.ac.databaseexplorer.common.component.csv.impl.SarCpuCsvReader;
  import jp.ac.databaseexplorer.common.exception.ApplicationException;
  import jp.ac.databaseexplorer.common.exception.SystemException;
  import lombok.NonNull;
  import lombok.RequiredArgsConstructor;
  import org.springframework.stereotype.Service;

  import java.util.Date;

  /**
   * CPU使用率を取得するサービスクラス
   */
  @Service
  @RequiredArgsConstructor
  public class CpuUsageService {

    /**
     * Reader
     */
    private final SarCpuCsvReader reader;

    /**
     * CPU使用率を取得する
     */
    public CpuUsageApiResponse getCpuUsage(@NonNull CpuUsageApiRequest request) throws ApplicationException {
      try {
        Date startTime = request.getStartTime();
        Date endTime = request.getEndTime();

        // インプット項目チェック
        if (request.getStartTime().after(request.getEndTime())) {
          throw new ApplicationException("", "", new Exception());
        }

        CpuUsageData[] cpuUsageData = reader.read(startTime, endTime).stream()
            .map(record -> new CpuUsageData(record.getTimestamp(), 100.0 - record.getIdle()))
            .toArray(CpuUsageData[]::new);
        return new CpuUsageApiResponse(startTime, endTime, cpuUsageData);
      } catch (SystemException se) {
        throw new ApplicationException("APP-00004", "CPU使用率取得処理でシステム例外が発生しました", se);
      } catch (ApplicationException ae) {
        throw new ApplicationException("APP-00005", "CPU使用率取得処理で業務例外が発生しました。", ae);
      } catch (Exception e) {
        throw new ApplicationException("APP-00006", "CPU使用率取得処理で予期せぬ例外が発生しました。", e);
      }
    }
  }
  ```

## 生成されたテストコード
  ```java
  package jp.ac.databaseexplorer.api.service.visualization;

  import jp.ac.databaseexplorer.api.model.visualization.CpuUsageApiRequest;
  import jp.ac.databaseexplorer.api.model.visualization.CpuUsageApiResponse;
  import jp.ac.databaseexplorer.api.model.visualization.CpuUsageData;
  import jp.ac.databaseexplorer.common.component.csv.impl.SarCpuCsvReader;
  import jp.ac.databaseexplorer.common.exception.ApplicationException;
  import jp.ac.databaseexplorer.common.exception.SystemException;
  import jp.ac.databaseexplorer.storage.visualization.SarCpu;
  import org.junit.jupiter.api.BeforeEach;
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.extension.ExtendWith;
  import org.mockito.Mock;
  import org.mockito.junit.jupiter.MockitoExtension;

  import java.util.Date;
  import java.util.List;

  import static org.junit.jupiter.api.Assertions.assertEquals;
  import static org.junit.jupiter.api.Assertions.assertThrows;
  import static org.mockito.Mockito.when;

  @ExtendWith(MockitoExtension.class)
  public class CpuUsageServiceTest {

    @Mock
    private SarCpuCsvReader reader;

    private CpuUsageService service;

    @BeforeEach
    public void setUp() {
      service = new CpuUsageService(reader);
    }

    @Test
    public void testGetCpuUsage() throws ApplicationException, SystemException {
      Date startTime = new Date();
      Date endTime = new Date();

      CpuUsageApiRequest request = new CpuUsageApiRequest(startTime, endTime);

      SarCpu sarCpu = new SarCpu();
      sarCpu.setTimestamp(startTime);
      sarCpu.setIdle(20.0);

      when(reader.read(startTime, endTime)).thenReturn(List.of(sarCpu));

      CpuUsageApiResponse response = service.getCpuUsage(request);

      assertEquals(startTime, response.getStartTime());
      assertEquals(endTime, response.getEndTime());
      CpuUsageData[] data = response.getCpuUsageData();
      assertEquals(1, data.length);
      assertEquals(startTime, data[0].getTimestamp());
      assertEquals(80.0, data[0].getUsage());
    }

    @Test
    public void testGetCpuUsageThrowsApplicationException() throws ApplicationException, SystemException {
      Date startTime = new Date();
      Date endTime = new Date(startTime.getTime() - 1); // endTime is before startTime

      CpuUsageApiRequest request = new CpuUsageApiRequest(startTime, endTime);

      assertThrows(ApplicationException.class, () -> service.getCpuUsage(request));
    }

    @Test
    public void testGetCpuUsageThrowsSystemException() throws ApplicationException, SystemException {
      Date startTime = new Date();
      Date endTime = new Date();

      CpuUsageApiRequest request = new CpuUsageApiRequest(startTime, endTime);

      when(reader.read(startTime, endTime)).thenThrow(SystemException.class);

      assertThrows(ApplicationException.class, () -> service.getCpuUsage(request));
    }
  }
  ```

# 参考情報
(*1) https://openai.com/blog/chatgpt
