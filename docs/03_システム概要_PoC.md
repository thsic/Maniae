# システム概要 (PoC版)

広島県内におけるバスのリアルタイム到着情報および過去の到着実績を提供し、利用者のバス利用体験を向上させるアプリケーションの概念実証（PoC）版です。

# PoCにおけるこだわり

-   **迅速な情報提供**: 利用者が求めるバスの運行情報を、最小限の操作で素早く表示することに重点を置きます。
-   **シンプルな操作性**: 直感的で分かりやすいインターフェースを提供し、誰でも簡単に利用できるようにします。

# 設計思想

-   **ドメイン駆動設計 (DDD)**: ビジネスの関心事をコードの中心に据え、複雑なドメインの問題を解決します。
-   **オニオンアーキテクチャ**: 関心の分離を徹底し、テスト容易性、保守性、拡張性の高いシステムを構築します。
-   **疎結合・高凝集**: 各モジュールの独立性を高め、変更に強く柔軟なシステムを目指します。

# アプリケーション方針 (PoC)

-   **シンプルなデザイン**: 利用者が情報に集中できる、クリーンでミニマルなデザインを採用します。
-   **即時表示**: アプリケーション起動後、迅速にバス情報を表示します。
-   **PoCにおけるコア機能**:
    -   リアルタイムバス停到着情報 (広島電鉄・広島バス)
    -   過去のバス停到着情報 (広島電鉄・広島バス)
    -   バスの遅延情報表示
-   **PoCでは提供しない機能**:
    -   運休情報
    -   バス路線検索
    -   ログイン機能
    -   乗り換え案内
    -   バスのリアルタイム位置情報（地図表示など）
    -   お気に入り機能

# 開発方針 (PoC)

-   **シンプルさ**: PoCの範囲に機能を絞り、複雑性を排除します。
-   **迅速な開発**: スピーディなイテレーションを重視し、早期にフィードバックを得ることを目指します。
-   **柔軟性**: 将来的な拡張を考慮しつつも、PoC段階ではコア機能の実装に集中します。
-   **段階的構築**: 「小さく始めて育てる」アプローチで、PoCの成功を足がかりに段階的に機能を拡張します。
-   **ライブラリ依存の最小化**: コアなロジックはフレームワークやライブラリへの過度な依存を避け、本質的な価値に集中します。（捨てやすく作る）

# 技術スタック (PoC)

## 共通基盤
-   Cursor (開発環境)
-   Github (バージョン管理)
-   Zod (型バリデーション)
-   Neverthrow (エラーハンドリング)

## CI/CD
-   Github Actions (自動テスト・デプロイ)
-   Playwright (E2Eテスト)

## フロントエンド
-   Cloudflare Pages (ホスティング)
-   PWA (プログレッシブウェブアプリ)
-   TypeScript
-   Angular
-   Prettier (コードフォーマッター)

## バックエンド
-   Cloudflare Workers (サーバーレス)
-   D1 (データベース - MySQL互換)
-   TypeScript
-   Hono (Webフレームワーク)
-   Vitest (ユニットテスト)
-   Biome (リンター、フォーマッター)
-   Drizzle (ORM)
-   fflate (圧縮・解凍ライブラリ)

## データソース
-   GTFS-JP (静的バス情報)
-   GTFS-RT (リアルタイムバス情報)
-   対象事業者 (PoC):
    -   広島電鉄株式会社
    -   広島バス株式会社
-   利用するGTFS-RTフィード (PoC):
    -   TripUpdate (車両の遅延や到着予測)

# コーディング規約・プラクティス

## テスト駆動開発 (TDD)
-   **テスト先行**: 実装前にテストコードを記述します。
-   **テストは仕様**: テストを動作する仕様書として扱います。
-   **反復的な開発**: 小さな単位で実装とテストを繰り返し、品質を高めます。
-   **継続的なリファクタリング**: Red-Green-Refactorサイクルを遵守し、コードの健全性を保ちます。

## リファクタリング
-   **適切なタイミング**: 一つのまとまった機能単位の実装完了後など、定期的に実施します。
-   **責務の明確化**: クラスやメソッドの責務が単一かつ明確であるかを確認します。
-   **命名の適切性**: 変数名、関数名、クラス名などがドメインの概念を正確に表しているかを見直します。
-   **ドメインモデル貧血症の回避**: ドメインオブジェクトが単なるデータコンテナではなく、振る舞いを持つように設計します。
-   **テストの網羅性**: テストケースが主要なロジックや境界値をカバーしているかを確認します。

## テスト戦略
-   **カバレッジ目標**: C0/C1カバレッジ85%以上を目指しますが、数値よりもテストの質を重視します。
-   **重点的なテスト対象**: Value ObjectとEntityはカバレッジ100%を目指します。
-   **テストフレームワーク**:
    -   フロントエンド: Angular標準テストツール (Jasmine/Karmaなど)
    -   バックエンド: Vitest
    -   E2Eテスト: Playwright

### 型定義の例 (ブランデッド型)
```typescript
// ブランデッド型により、プリミティブ型にドメインの意味づけを行い型安全性を向上
type Branded<T, B> = T & { _brand: B };

// 例: 金額を表す型 (単なるnumber型ではなく、Money型として区別)
type Money = Branded<number, "Money">;

// 例: メールアドレスを表す型
type Email = Branded<string, "Email">;
```

## エラーハンドリング
-   **アーリーリターン**: 関数の早い段階で不正な状態を検知し、処理を中断します。
-   **Result型**: 成功と失敗の両方の可能性を型で明示的に表現し、堅牢なエラー処理を促します。

```typescript
import { ok, err, Result } from 'neverthrow';

// 何らかの処理を行う関数
function processData(data: unknown): Result<string, Error> {
  if (typeof data !== 'string') {
    return err(new Error('Invalid data type'));
  }
  if (data.trim() === '') {
    return err(new Error('Data cannot be empty'));
  }
  // 成功した場合は ok でラップして返す
  return ok(`Processed: ${data}`);
}

// Result型の使用例
const result1 = processData("hello");
if (result1.isOk()) {
  console.log(result1.value); // "Processed: hello"
} else {
  console.error(result1.error.message);
}

const result2 = processData(123); // Invalid data type
if (result2.isErr()) {
  console.error(result2.error.message);
}
```

## 依存性注入 (DI)
-   **抽象への依存**: 具体的な実装ではなく、抽象（インターフェースや抽象クラス）に依存します。
-   **DIコンテナの活用**: 必要に応じてDIコンテナを利用し、依存関係の管理を容易にします。
-   **定義場所**: 抽象はドメイン層またはユースケース層（アプリケーション層）に定義します。

## クラス設計
-   **単一責任の原則**: クラスは一つの明確な責務を持ちます。
-   **クラスの分割**: クラスが肥大化する場合は、適切に分割します。
-   **インターフェースの最小化**: クラスの公開メソッドは必要最小限（再構築メソッドを除く場合は1つが目安）にします。
-   **具体的な命名**: クラス名は、そのクラスが持つ具体的な責務を明確に表す名詞、または「<動詞><名詞>」のような動詞句（例: `RouteFinder`, `ArrivalNotifier`, `TicketIssuer`）を使用します。`OrderService` や `UserService` のような、単に `Service` を接尾辞とする曖昧な命名は避け、より具体的な役割を示すようにします。
-   **ユビキタス言語の活用**: クラス名やメソッド名は、ドメインエキスパートと開発チームが共有するユビキタス言語に基づいて命名します。
-   **責務に応じた命名パターン例**:
    -   特定の集約やエンティティの永続化を担当する場合: `XxxRepository` (例: `StopRepository`)
    -   複雑なオブジェクトの生成を担当する場合: `XxxFactory` (例: `TripFactory`)
    -   特定のビジネスプロセスやユースケースを実行する場合: ユースケースを具体的に表す動詞句 (例: `FindRealtimeArrivals`, `UpdateTripStatus`) または `XxxUseCase` (例: `GetBusStopScheduleUseCase`)
    -   外部システムとの連携や通知を行う場合: `XxxAdapter`, `XxxClient`, `XxxNotifier` (例: `GtfsRtClient`, `DelayNotifier`)

## ドメイン層・ユースケース層（アプリケーション層）の指針
-   **関数型プログラミングの活用**: 不変性や純粋関数などの概念を取り入れ、見通しの良いコードを目指します。
-   **コードの表現力**: コード自体がドキュメントの役割を果たすよう、命名や構造を工夫します。
-   **利用ライブラリの制限**: ドメインロジックの純粋性を保つため、原則としてZodとNeverthrow以外の外部ライブラリへの依存を避けます。

```typescript
import { z } from 'zod';
import { ok, err, Result } from 'neverthrow';

// 値オブジェクトのスキーマ定義 (Zod)
const StopIdSchema = z.string().min(1).max(20);
type StopId = z.infer<typeof StopIdSchema>;

// 値オブジェクトのファクトリ関数 (バリデーションとResult型を組み合わせる)
function createStopId(value: unknown): Result<StopId, z.ZodError> {
  const validationResult = StopIdSchema.safeParse(value);
  if (!validationResult.success) {
    return err(validationResult.error);
  }
  return ok(validationResult.data);
}

// ドメインサービスやエンティティのメソッド内での利用例
class StopQuery {
  public getStopInfo(rawStopId: unknown): Result<string, Error> {
    const stopIdResult = createStopId(rawStopId);
    if (stopIdResult.isErr()) {
      // Zodのエラーをアプリケーション独自のエラーに変換しても良い
      return err(new Error(`Invalid StopId: ${stopIdResult.error.message}`));
    }
    const stopId = stopIdResult.value;
    // stopId を使った処理...
    return ok(`Information for stop: ${stopId}`);
  }
}

const query = new StopQuery();
const infoResult = query.getStopInfo("valid-stop-id");
if (infoResult.isOk()) {
  console.log(infoResult.value);
} else {
  console.error(infoResult.error.message);
}

const errorResult = query.getStopInfo("this is way too long for a stop id");
if (errorResult.isErr()) {
  console.error(errorResult.error.message); // "Invalid StopId: ..."
}
```

# PoCにおける主要機能

## バッチ処理
### GTFS-JPデータ更新
-   **実行タイミング**: 現行のFeedVersionの有効期限が切れる1週間前のAM4:00に自動実行。
-   **処理内容**: 最新のGTFS-JPデータを取得し、データベースを更新。

### GTFS-RTデータ取得・更新
-   **実行頻度**: 30秒に1回実行。
-   **処理内容**: 対象事業者のGTFS-RTフィード（TripUpdate）を取得し、バスの運行情報（遅延、到着予測など）を更新し、必要に応じて状態を遷移させます。

## API
-   **目的**: Webフロントエンドからのバス情報に関する問い合わせに対応します。
-   **認証・アクセス制限 (PoC)**:
    -   ログイン機能は実装しません。
    -   Cloudflareの機能を利用したオリジンベースのアクセス制御および基本的なレート制限を想定します。

```typescript
import { Hono } from 'hono';
import { bearerAuth } from 'hono/bearer-auth'; // 例: 簡単なBearer認証

// 環境変数などからトークンを取得 (実際には安全な方法で管理)
const TOKEN = 'your-secure-token';

const app = new Hono();

// 簡単な認証ミドルウェア (PoCの範囲や要件に応じて調整)
// app.use('/api/secure/*', bearerAuth({ token: TOKEN }));

// ルート定義の例
app.get('/', (c) => c.text('Welcome to Maniae App API!'));

// バス停情報の取得API (例)
// GET /api/stops/:stopId/schedule
app.get('/api/stops/:stopId/schedule', (c) => {
  const stopId = c.req.param('stopId');
  // ここで stopId を使ってバス停のスケジュールを取得するロジックを呼び出す
  // (例: StopService.getSchedule(stopId) など)
  const scheduleData = { // ダミーデータ
    stopId: stopId,
    arrivals: [
      { line: 'A1', time: '10:00', delay: 0 },
      { line: 'B2', time: '10:05', delay: 5 },
    ],
  };
  return c.json(scheduleData);
});

// 過去の到着実績API (例)
// GET /api/stops/:stopId/history?date=YYYY-MM-DD
app.get('/api/stops/:stopId/history', (c) => {
  const stopId = c.req.param('stopId');
  const date = c.req.query('date'); // クエリパラメータから日付を取得

  // バリデーションやエラーハンドリングをここで行う
  if (!date) {
    return c.json({ error: 'Date query parameter is required' }, 400);
  }

  const historyData = { // ダミーデータ
    stopId: stopId,
    date: date,
    records: [
      { line: 'A1', scheduled: '09:00', actual: '09:02' },
      { line: 'C3', scheduled: '09:15', actual: '09:15' },
    ],
  };
  return c.json(historyData);
});

export default app; // Cloudflare Workers で Hono を使う場合の一般的なエクスポート
```

## Webフロントエンド
-   **中心機能**: バス停の運行情報を中心とした情報表示。
-   **画面構成 (PoC)**:
    -   **メイン画面**:
        -   指定したバス停のリアルタイム運行情報（到着予定、遅延など）を表示します。
        -   過去の到着実績も確認できるようにします。

# データベース設計 (PoC)
-   **基本方針**: 将来的な拡張性を考慮しつつ、PoCで必要な最小限のテーブル構成とします。詳細はドメインモデル図を参照。

# CI/CD戦略 (PoC)
-   **ブランチ戦略**:
    -   `main`: 本番リリース用ブランチ (PoCではCloudflare Pagesへのデプロイ先)
    -   `develop`: 開発のベースとなるブランチ
    -   `feature/*`: 各機能開発用ブランチ
-   **Pull Request時 (developブランチ、mainブランチへのマージ前)**:
    -   LintおよびFormatチェック (Biome, Prettier)
    -   ビルド (フロントエンド・バックエンド)
    -   ユニットテスト (Vitest)
    -   E2Eテスト (Playwright, プレビュー環境に対して実行)
    -   依存性チェック (npm auditなど)
-   **マージ時 (`develop`ブランチ)**:
    -   PR時のジョブに加えて、Staging/Preview環境への自動デプロイ (Cloudflare Pages/Workers)
    -   デプロイ後の基本的なスモークテスト
    -   開発チームへの通知
-   **マージ時 (`main`ブランチ - PoCにおける本番デプロイ)**:
    -   PR時のジョブ (E2Eテスト含む)
    -   Production環境への自動デプロイ (Cloudflare Pages/Workers)
    -   デプロイ後のスモークテスト
    -   関係者への通知
-   **PoCにおけるポイント**:
    -   **小さく開始**: まずは基本的なCI/CDパイプラインを構築し、徐々に改善します。
    -   **実行速度**: 各ジョブの実行時間を意識し、開発サイクルを高速に保ちます。
    -   **Secrets管理**: APIキーなどの機密情報はGithub ActionsのSecrets機能で安全に管理します。
    -   **ロールバック計画**: (PoCでは手動ロールバックを想定しつつ、将来的には自動化も視野)
    -   **DBマイグレーション**: D1とDrizzle Migrationsを連携させ、安全なスキーマ変更を実現します。

# ディレクトリ構成案

## フロントエンド (Angular)
-   `src/app/`
    -   `core/`: コア機能、サービス、ガードなど (旧 `domain` の一部責務も含む可能性)
    -   `features/`: 機能ごとのモジュール (例: `bus-schedule`, `stop-info`)
        -   `[feature-name]/`
            -   `components/`
            -   `services/` (そのフィーチャー固有のサービス)
            -   `models/` (そのフィーチャー固有のデータモデル)
    -   `shared/`: 共有モジュール、コンポーネント、パイプ、ディレクティブなど
    -   `infrastructure/`: APIクライアントなど、外部システムとの連携部分
    -   `routes/` または `app-routing.module.ts`: ルーティング定義

## バックエンド (Hono on Cloudflare Workers)
-   `src/` (ルートまたは `api/` など任意のディレクトリ名)
    -   `domain/`: ドメインエンティティ、値オブジェクト、ドメインサービス
    -   `application/`: ユースケース、アプリケーションサービス
    -   `infrastructure/`: データベースアクセス(Drizzle)、外部APIクライアント、バッチ処理など
    -   `interfaces/` または `routes/` または `handlers/`: APIエンドポイント定義 (Honoのルーティング)
    -   `config/`: 環境設定など
    -   `lib/` または `shared/`: 共通ライブラリ、ユーティリティ関数

**備考**: 上記はあくまで一例です。プロジェクトの規模やチームの慣習に応じて調整してください。重要なのは、高凝集（関連性の高いものを集める）かつ疎結合（依存関係を減らす）な構造を意識することです。
