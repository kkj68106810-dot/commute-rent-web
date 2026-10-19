# Contributing Guide / コントリビューションガイド

## 1. Commit Message Rules / コミットメッセージ規約

### Format / 形式

```
<type>: <summary>

- detail 1 (optional)
- detail 2 (optional)
```

- 첫 줄은 영어로, 50자 이내 / 1行目は英語、50文字以内
- 날짜는 넣지 않음 (git이 기록) / 日付は不要（gitが記録）
- 세부 내용이 있으면 한 줄 띄우고 목록으로 / 詳細は1行空けて箇条書き

### Types / 種類

| Type | 한국어 | 日本語 | Example |
|---|---|---|---|
| `feat` | 새 기능 추가 | 新機能の追加 | `feat: add station search API` |
| `fix` | 버그·잘못된 부분 수정 (스키마 수정 포함) | バグ・誤りの修正（スキーマ修正含む） | `fix: update stations table and SQL script` |
| `refactor` | 동작 변경 없이 코드 구조·이름 정리 | 動作を変えないコード整理・命名変更 | `refactor: rename yaching to commute-rent` |
| `docs` | README·주석 등 문서만 변경 | README・コメントなどドキュメントのみ | `docs: update README for v1.3 schema` |
| `chore` | 설정·의존성·빌드 등 기능 외 작업 | 設定・依存関係・ビルドなど | `chore: move secrets to env vars` |
| `test` | 테스트 코드 추가·수정 | テストコードの追加・修正 | `test: add Dijkstra service tests` |

### How to choose / 迷ったとき

- 사용자가 새로 쓸 수 있는 게 생김 → `feat` / ユーザーが新しく使える機能 → `feat`
- 원래 틀린 걸 바로잡음 → `fix` / 誤りを直した → `fix`
- 결과는 같고 코드만 정리 → `refactor` / 結果は同じでコード整理のみ → `refactor`
- 종류가 섞이면 커밋을 나눈다 / 種類が混ざる場合はコミットを分ける

### Example / 例

```
fix: update stations schema to v1.3

- station_name: VARCHAR(20) -> VARCHAR(50)
- add is_major_workplace BOOLEAN NOT NULL DEFAULT FALSE
```

## 2. Secrets / 機密情報

- 비밀번호·API 키·JWT secret은 커밋하지 않고 환경변수로 관리
- パスワード・APIキー・JWT secretはコミットせず環境変数で管理
