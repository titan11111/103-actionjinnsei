# LEARNINGS - 103-actionjinnsei

## 2026-10-28

- harness FAILの原因は3つ独立していた。
  1. `js/init.js` が起動時に `stateMachine.changeState(GameState.TITLE)` を呼んでいたが、初期状態が既にTITLEのため遷移表で拒否され、毎回 `console.error`（Invalid state transition: TITLE → TITLE）が出ていた。`stateMachine.reset()` に変更して解消。
  2. DOM順で `#statusBar` 内の `#pauseBtn` が `#startScreen` 内の `#startButton` より先に出現しており、harnessの「最初のbutton/canvasをタップ」チェックが非表示のpauseBtnを掴んでタイムアウトしていた。`#startScreen` を `#gameContainer` の先頭に移動して解消（CSSは全てposition:absolute/z-index指定済みのため見た目は変化なし）。
  3. `.startBtn` の `buttonPulse` アニメーションが `transform: scale()` で要素自体のバウンディングボックスを動かし続けており、Playwrightの「stable」判定が永久に成立せずタップが失敗していた。`transform`のscale変動を削除し、`box-shadow`だけで脈動を表現するよう変更（見た目のパルス効果はほぼ維持）。
- BGM9曲（計14MB）が `preload="auto"` で起動直後に一括フェッチされ、モバイル回線を想定すると過剰。`preload="none"` に変更し再生時に遅延ロードへ。harnessの通信量も14.10MB→1.96MBに減少。
- 本番URL (https://titan11111.github.io/103-actionjinnsei/) が古い単一HTML版のままだったのを発見。原因はこのゲーム専用のGitHubリポジトリ（`titan11111/103-actionjinnsei`）に対してローカルの`.git`がまだ無く、`publish.sh`でpushされていなかったため（親リポ`GitHub-game`側のgit同期チェックだけでは検出できない。配信は常にゲーム単位の別リポジトリ）。

### 次回に使う条件
- harnessの「タップ失敗」「element is not stable」が出たら、まずCSSの `transform` 系infiniteアニメーションを疑う（scale/translateが入っているとPlaywrightは永久に安定判定できない）。
- harnessの「タップ失敗」でDOM順も疑う：`button, [role="button"], canvas` の最初の一致要素が実際に押せる主要CTAか`grep -n`で確認する。
- 本番URLのハッシュ不一致は、親リポの`git status`/`git fetch`だけでは判定できない。ゲームごとに別リポジトリがある前提で、そのゲームの`.git`の有無と`publish.sh`の実行履歴を先に確認する。
