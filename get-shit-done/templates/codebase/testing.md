# テストパターンテンプレート

`.planning/codebase/TESTING.md` 用テンプレート - テストフレームワークとパターンを記録します。

**目的:** テストの書き方と実行方法を文書化します。既存のパターンに合ったテストを追加するためのガイド。

---

## ファイルテンプレート

```markdown
# テストパターン

**分析日:** [YYYY-MM-DD]

## テストフレームワーク

**ランナー:**
- [フレームワーク: 例: "Jest 29.x", "Vitest 1.x"]
- [設定: 例: "プロジェクトルートのjest.config.js"]

**アサーションライブラリ:**
- [ライブラリ: 例: "組み込みexpect", "chai"]
- [マッチャー: 例: "toBe, toEqual, toThrow"]

**実行コマンド:**
```bash
[例: "npm test"または"npm run test"]              # すべてのテストを実行
[例: "npm test -- --watch"]                       # ウォッチモード
[例: "npm test -- path/to/file.test.ts"]          # 単一ファイル
[例: "npm run test:coverage"]                     # カバレッジレポート
```

## テストファイルの構成

**場所:**
- [パターン: 例: "ソースファイルと並んで*.test.ts"]
- [代替: 例: "__tests__/ディレクトリ"または"別のtests/ツリー"]

**命名:**
- [ユニットテスト: 例: "module-name.test.ts"]
- [統合テスト: 例: "feature-name.integration.test.ts"]
- [E2E: 例: "user-flow.e2e.test.ts"]

**構造:**
```
[実際のディレクトリパターンを表示、例:
src/
  lib/
    utils.ts
    utils.test.ts
  services/
    user-service.ts
    user-service.test.ts
]
```

## テスト構造

**スイートの構成:**
```typescript
[実際に使用されているパターンを表示、例:

describe('ModuleName', () => {
  describe('functionName', () => {
    it('should handle success case', () => {
      // arrange
      // act
      // assert
    });

    it('should handle error case', () => {
      // テストコード
    });
  });
});
]
```

**パターン:**
- [セットアップ: 例: "共有セットアップにbeforeEach、beforeAllは避ける"]
- [ティアダウン: 例: "クリーンアップとモック復元にafterEach"]
- [構造: 例: "arrange/act/assertパターン必須"]

## モッキング

**フレームワーク:**
- [ツール: 例: "Jest組み込みモッキング", "Vitest vi", "Sinon"]
- [インポートモッキング: 例: "ファイル先頭でvi.mock()"]

**パターン:**
```typescript
[実際のモッキングパターンを表示、例:

// 外部依存関係のモック
vi.mock('./external-service', () => ({
  fetchData: vi.fn()
}));

// テスト内でのモック
const mockFetch = vi.mocked(fetchData);
mockFetch.mockResolvedValue({ data: 'test' });
]
```

**モックすべきもの:**
- [例: "外部API、ファイルシステム、データベース"]
- [例: "時間/日付（vi.useFakeTimersを使用）"]
- [例: "ネットワーク呼び出し（モックfetchを使用）"]

**モックすべきでないもの:**
- [例: "純粋関数、ユーティリティ"]
- [例: "内部ビジネスロジック"]

## フィクスチャとファクトリ

**テストデータ:**
```typescript
[テストデータ作成パターンを表示、例:

// ファクトリパターン
function createTestUser(overrides?: Partial<User>): User {
  return {
    id: 'test-id',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  };
}

// フィクスチャファイル
// tests/fixtures/users.ts
export const mockUsers = [/* ... */];
]
```

**場所:**
- [例: "共有フィクスチャはtests/fixtures/"]
- [例: "ファクトリ関数はテストファイルまたはtests/factories/内"]

## カバレッジ

**要件:**
- [目標: 例: "80%行カバレッジ", "特定の目標なし"]
- [施行: 例: "CIが80%未満でブロック", "認識のためのカバレッジのみ"]

**設定:**
- [ツール: 例: "--coverageフラグによる組み込みカバレッジ"]
- [除外: 例: "*.test.ts、設定ファイルを除外"]

**カバレッジの確認:**
```bash
[例: "npm run test:coverage"]
[例: "open coverage/index.html"]
```

## テストの種類

**ユニットテスト:**
- [スコープ: 例: "単一の関数/クラスを分離してテスト"]
- [モッキング: 例: "すべての外部依存関係をモック"]
- [速度: 例: "テストごとに1秒未満で実行必須"]

**統合テスト:**
- [スコープ: 例: "複数のモジュールを組み合わせてテスト"]
- [モッキング: 例: "外部サービスをモック、内部モジュールは実物を使用"]
- [セットアップ: 例: "テストデータベースを使用、データをシード"]

**E2Eテスト:**
- [フレームワーク: 例: "E2E用Playwright"]
- [スコープ: 例: "完全なユーザーフローをテスト"]
- [場所: 例: "ユニットテストとは別のe2e/ディレクトリ"]

## 一般的なパターン

**非同期テスト:**
```typescript
[パターンを表示、例:

it('should handle async operation', async () => {
  const result = await asyncFunction();
  expect(result).toBe('expected');
});
]
```

**エラーテスト:**
```typescript
[パターンを表示、例:

it('should throw on invalid input', () => {
  expect(() => functionCall()).toThrow('error message');
});

// 非同期エラー
it('should reject on failure', async () => {
  await expect(asyncCall()).rejects.toThrow('error message');
});
]
```

**スナップショットテスト:**
- [使用法: 例: "Reactコンポーネントのみ"または"使用していない"]
- [場所: 例: "__snapshots__/ディレクトリ"]

---

*テスト分析: [日付]*
*テストパターンが変更された際に更新*
```

<good_examples>
```markdown
# テストパターン

**分析日:** 2025-01-20

## テストフレームワーク

**ランナー:**
- Vitest 1.0.4
- 設定: プロジェクトルートのvitest.config.ts

**アサーションライブラリ:**
- Vitest組み込みexpect
- マッチャー: toBe, toEqual, toThrow, toMatchObject

**実行コマンド:**
```bash
npm test                              # すべてのテストを実行
npm test -- --watch                   # ウォッチモード
npm test -- path/to/file.test.ts     # 単一ファイル
npm run test:coverage                 # カバレッジレポート
```

## テストファイルの構成

**場所:**
- ソースファイルと並んで*.test.ts
- 別のtests/ディレクトリなし

**命名:**
- すべてのテストにunit-name.test.ts
- ファイル名でユニット/統合の区別なし

**構造:**
```
src/
  lib/
    parser.ts
    parser.test.ts
  services/
    install-service.ts
    install-service.test.ts
  bin/
    install.ts
    （テストなし - CLI経由の統合テスト）
```

## テスト構造

**スイートの構成:**
```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';

describe('ModuleName', () => {
  describe('functionName', () => {
    beforeEach(() => {
      // 状態をリセット
    });

    it('should handle valid input', () => {
      // arrange
      const input = createTestInput();

      // act
      const result = functionName(input);

      // assert
      expect(result).toEqual(expectedOutput);
    });

    it('should throw on invalid input', () => {
      expect(() => functionName(null)).toThrow('Invalid input');
    });
  });
});
```

**パターン:**
- テストごとのセットアップにbeforeEachを使用、beforeAllは避ける
- モック復元にafterEachを使用: vi.restoreAllMocks()
- 複雑なテストでは明示的なarrange/act/assertコメント
- テストごとに1つのアサーションフォーカス（複数のexpectは可）

## モッキング

**フレームワーク:**
- Vitest組み込みモッキング（vi）
- テストファイル先頭でvi.mock()によるモジュールモッキング

**パターン:**
```typescript
import { vi } from 'vitest';
import { externalFunction } from './external';

// モジュールをモック
vi.mock('./external', () => ({
  externalFunction: vi.fn()
}));

describe('test suite', () => {
  it('mocks function', () => {
    const mockFn = vi.mocked(externalFunction);
    mockFn.mockReturnValue('mocked result');

    // モックされた関数を使用するテストコード

    expect(mockFn).toHaveBeenCalledWith('expected arg');
  });
});
```

**モックすべきもの:**
- ファイルシステム操作（fs-extra）
- 子プロセス実行（child_process.exec）
- 外部APIコール
- 環境変数（process.env）

**モックすべきでないもの:**
- 内部の純粋関数
- シンプルなユーティリティ（文字列操作、配列ヘルパー）
- TypeScriptの型

## フィクスチャとファクトリ

**テストデータ:**
```typescript
// テストファイル内のファクトリ関数
function createTestConfig(overrides?: Partial<Config>): Config {
  return {
    targetDir: '/tmp/test',
    global: false,
    ...overrides
  };
}

// tests/fixtures/内の共有フィクスチャ
// tests/fixtures/sample-command.md
export const sampleCommand = `---
description: Test command
---
Content here`;
```

**場所:**
- ファクトリ関数: 使用箇所近くのテストファイル内で定義
- 共有フィクスチャ: tests/fixtures/（複数ファイルのテストデータ用）
- モックデータ: シンプルな場合はテスト内にインライン、複雑な場合はファクトリ

## カバレッジ

**要件:**
- 施行されるカバレッジ目標なし
- 認識のためにカバレッジを追跡
- クリティカルパスに焦点（パーサー、サービスロジック）

**設定:**
- c8経由のVitestカバレッジ（組み込み）
- 除外: *.test.ts, bin/install.ts, 設定ファイル

**カバレッジの確認:**
```bash
npm run test:coverage
open coverage/index.html
```

## テストの種類

**ユニットテスト:**
- 単一の関数を分離してテスト
- すべての外部依存関係をモック（fs, child_process）
- 高速: 各テスト100ms未満
- 例: parser.test.ts, validator.test.ts

**統合テスト:**
- 複数のモジュールを組み合わせてテスト
- 外部境界のみモック（ファイルシステム、プロセス）
- 例: install-service.test.ts（サービス + パーサーをテスト）

**E2Eテスト:**
- 現在は使用していない
- CLI統合テストは手動で実施

## 一般的なパターン

**非同期テスト:**
```typescript
it('should handle async operation', async () => {
  const result = await asyncFunction();
  expect(result).toBe('expected');
});
```

**エラーテスト:**
```typescript
it('should throw on invalid input', () => {
  expect(() => parse(null)).toThrow('Cannot parse null');
});

// 非同期エラー
it('should reject on file not found', async () => {
  await expect(readConfig('invalid.txt')).rejects.toThrow('ENOENT');
});
```

**ファイルシステムのモッキング:**
```typescript
import { vi } from 'vitest';
import * as fs from 'fs-extra';

vi.mock('fs-extra');

it('mocks file system', () => {
  vi.mocked(fs.readFile).mockResolvedValue('file content');
  // テストコード
});
```

**スナップショットテスト:**
- このコードベースでは使用していない
- 明確性のために明示的なアサーションを優先

---

*テスト分析: 2025-01-20*
*テストパターンが変更された際に更新*
```
</good_examples>

<guidelines>
**TESTING.mdに含めるもの:**
- テストフレームワークとランナーの設定
- テストファイルの場所と命名パターン
- テスト構造（describe/it、beforeEachパターン）
- モッキングアプローチと例
- フィクスチャ/ファクトリパターン
- カバレッジ要件
- テストの実行方法（コマンド）
- 実際のコードでの一般的なテストパターン

**ここに含めないもの:**
- 個別のテストケース（実際のテストファイルに委ねる）
- 技術選定（STACK.mdの範囲）
- CI/CDセットアップ（デプロイドキュメントの範囲）

**このテンプレートを記入する際:**
- package.jsonスクリプトでテストコマンドを確認
- テスト設定ファイルを見つける（jest.config.js, vitest.config.ts）
- 3〜5の既存テストファイルを読んでパターンを特定
- tests/またはtest-utils/のテストユーティリティを探す
- カバレッジ設定を確認
- 理想的なパターンではなく、実際に使用されているパターンを文書化

**フェーズ計画で役立つ場面:**
- 新機能の追加（マッチするテストを書く）
- リファクタリング（テストパターンを維持）
- バグ修正（リグレッションテストを追加）
- 検証アプローチの理解
- テストインフラストラクチャのセットアップ

**分析アプローチ:**
- package.jsonでテストフレームワークとスクリプトを確認
- テスト設定ファイルでカバレッジ、セットアップを読む
- テストファイルの構成を調査（併置か分離か）
- 5つのテストファイルでパターンを確認（モッキング、構造、アサーション）
- テストユーティリティ、フィクスチャ、ファクトリを探す
- テストの種類に注目（ユニット、統合、E2E）
- テスト実行コマンドを文書化
</guidelines>
