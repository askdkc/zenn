---
title: "Claude CodeのハーネスでChatGPTモデルを使う"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [ai, codex, claude]
published: true
---
# Claude CodeのハーネスでChatGPTモデルを使う
XでChatGPTをClaude Codeのハーネスで利用すると高性能になるとのポストを見かけて、それをmacOSで実現した時の手順を書いておきます。

- 元ネタ
![](https://static.zenn.studio/user-upload/1f96ed8bbbb2-20260725.png)

https://x.com/OmedVibeCodes/status/2080348655039500703

個人的にmacOSで動作するDockerが遅くて嫌いなので、Appleのオフィシャルなコンテナサービス `container` を使う手順にしてあります。

## 概要

Docker Desktopを使わず、macOSのApple `container` CLIでCLIProxyAPIを実行し、 Codex OAuth経由でGPT-5.6 SolをClaude Codeのバックエンドとして利用する。

    Claude Code
        ↓ Anthropic Messages互換API
    CLIProxyAPI
        ↓ Codex OAuth
    GPT-5.6 Sol / GPT-5.6 Terra

Apple `container` が自動設定したDNS `192.168.64.1` が動かない場合があるのでDNSはCloudflareを指定

``` zsh
--dns 1.1.1.1
```

## 前提条件

- macOS
- Apple `container` CLI
- Claude Code
- ChatGPTアカウント
- `curl`
- `jq`
- `openssl`

確認:

``` zsh
container --version
container system status
claude --version
curl --version
jq --version
```

`container version` ではなく `container --version` を使う。

Apple `container` が停止している場合:

``` zsh
container system start
container system status
```

## ディレクトリを作成する

``` zsh
mkdir -p \
  "$HOME/.config/cliproxy" \
  "$HOME/.local/share/cliproxy/auth" \
  "$HOME/.local/share/cliproxy/plugins" \
  "$HOME/.local/bin"

chmod 700 \
  "$HOME/.config/cliproxy" \
  "$HOME/.local/share/cliproxy" \
  "$HOME/.local/share/cliproxy/auth" \
  "$HOME/.local/share/cliproxy/plugins"
```

## ローカルAPIトークンを生成する

既存トークンを上書きしない。

``` zsh
if [[ ! -s "$HOME/.config/cliproxy/token" ]]; then
  openssl rand -hex 32 > "$HOME/.config/cliproxy/token"
fi

chmod 600 "$HOME/.config/cliproxy/token"
```

確認:

``` zsh
cat "$HOME/.config/cliproxy/token"
```

このトークンはClaude CodeからCLIProxyAPIへの接続認証に使用する。

## CLIProxyAPI設定ファイルを作成する

``` zsh
TOKEN="$(cat "$HOME/.config/cliproxy/token")"

cat > "$HOME/.config/cliproxy/config.yaml" <<YAML
host: "0.0.0.0"
port: 8317

tls:
  enable: false

auth-dir: "/root/.cli-proxy-api"

api-keys:
  - "$TOKEN"

remote-management:
  allow-remote: false
  secret-key: ""
  disable-control-panel: true

debug: false
logging-to-file: false
usage-statistics-enabled: false

payload:
  override:
    - models:
        - name: "gpt-5.6-sol"
          protocol: "codex"
      params:
        "reasoning.effort": "xhigh"
YAML

chmod 600 "$HOME/.config/cliproxy/config.yaml"
unset TOKEN
```

設定の要点:

- コンテナ内では `0.0.0.0:8317` で待ち受ける
- ホスト側は後で `127.0.0.1:8317` のみに公開する
- Codex OAuth認証情報は `/root/.cli-proxy-api` に保存する
- `gpt-5.6-sol` の推論強度を `xhigh` に固定する
- リモート管理画面は無効化する

## Apple containerでのbind mount

単一ファイルを直接bind mountすると、次のエラーになることがある。

    Error: path '/Users/.../.config/cliproxy/config.yaml' is not a directory

そのため次の形式は使用しない。

``` zsh
--mount "type=bind,source=$HOME/.config/cliproxy/config.yaml,target=/CLIProxyAPI/config.yaml"
```

設定ディレクトリ全体を `/config` にmountする。

``` zsh
--mount "type=bind,source=$HOME/.config/cliproxy,target=/config"
```

CLIProxyAPIには設定ファイルの場所を明示する。

``` zsh
--config /config/config.yaml
```

## CLIProxyAPIイメージを取得する

``` zsh
IMAGE='docker.io/eceasy/cli-proxy-api:latest'

container image pull "$IMAGE"
```

確認:

``` zsh
container image list
```

## DNS疎通を確認する

Apple `container` の既定DNSを使わず、外部DNSを明示する。（上手く通信できない時とかあるので）

``` zsh
container run --rm \
  --dns 1.1.1.1 \
  alpine:latest \
  nslookup auth.openai.com
```

正常ならIPアドレスが返る。

## Codex OAuthで認証する

``` zsh
IMAGE='docker.io/eceasy/cli-proxy-api:latest'

container run --rm -it \
  --name cliproxy-login \
  --dns 1.1.1.1 \
  -p 127.0.0.1:1455:1455 \
  --mount "type=bind,source=$HOME/.config/cliproxy,target=/config" \
  --mount "type=bind,source=$HOME/.local/share/cliproxy/auth,target=/root/.cli-proxy-api" \
  --mount "type=bind,source=$HOME/.local/share/cliproxy/plugins,target=/CLIProxyAPI/plugins" \
  "$IMAGE" \
  /CLIProxyAPI/CLIProxyAPI \
  --config /config/config.yaml \
  --no-browser \
  --codex-login
```

表示されたOpenAI OAuth URLをmacOSのブラウザで開く。（API課金じゃなくてサブスクが使えてお安い）

成功時:

    Codex authentication successful!

認証ファイルを確認する。

``` zsh
find "$HOME/.local/share/cliproxy/auth" \
  -maxdepth 2 \
  -type f \
  -ls
```

認証情報の保存先:

    ~/.local/share/cliproxy/auth

※このディレクトリをGitへ追加しないようにしてね。

## CLIProxyAPIを常駐起動する

既存の同名コンテナがある場合は削除する。

``` zsh
container delete --force cliproxy 2>/dev/null || true
```

起動:

``` zsh
IMAGE='docker.io/eceasy/cli-proxy-api:latest'

container run --detach \
  --name cliproxy \
  --dns 1.1.1.1 \
  --cpus 3 \
  --memory 3G \
  -p 127.0.0.1:8317:8317 \
  --mount "type=bind,source=$HOME/.config/cliproxy,target=/config" \
  --mount "type=bind,source=$HOME/.local/share/cliproxy/auth,target=/root/.cli-proxy-api" \
  --mount "type=bind,source=$HOME/.local/share/cliproxy/plugins,target=/CLIProxyAPI/plugins" \
  "$IMAGE" \
  /CLIProxyAPI/CLIProxyAPI \
  --config /config/config.yaml
```

ポートはmacOSのローカルホストだけへ公開する。

    127.0.0.1:8317 → container:8317

LANへ公開しない。

## 起動状態とログを確認する

``` zsh
container list --all
```

正常例:

    cliproxy ... running

ログ:

``` zsh
container logs cliproxy
```

正常時のログ例:

    API server started successfully on: 0.0.0.0:8317
    server clients and configuration updated: 1 clients

継続監視:

``` zsh
container logs -f cliproxy
```

## モデル一覧を確認する

``` zsh
PROXY_TOKEN="$(cat "$HOME/.config/cliproxy/token")"

curl -fsS http://127.0.0.1:8317/v1/models \
  -H "Authorization: Bearer $PROXY_TOKEN" |
  jq -r '.data[].id' |
  grep 'gpt-5\.6'
```

正常例:

    gpt-5.6-sol
    gpt-5.6-terra


`gpt-5.6-sol` が表示されない場合、Claude Code側の設定へ進まない。

## Anthropic Messages互換APIを確認する

``` zsh
PROXY_TOKEN="$(cat "$HOME/.config/cliproxy/token")"

curl -fsS http://127.0.0.1:8317/v1/messages \
  -H "Authorization: Bearer $PROXY_TOKEN" \
  -H 'anthropic-version: 2023-06-01' \
  -H 'content-type: application/json' \
  -d '{
    "model": "gpt-5.6-sol",
    "max_tokens": 32,
    "messages": [
      {
        "role": "user",
        "content": "Reply with exactly OK."
      }
    ]
  }' | jq
```

正常例:

``` json
{
  "type": "message",
  "role": "assistant",
  "model": "gpt-5.6-sol",
  "content": [
    {
      "type": "text",
      "text": "OK"
    }
  ]
}
```

ここが成功してからClaude Codeを接続する。

## Claude Codeを一時的に接続する

``` zsh
PROXY_TOKEN="$(cat "$HOME/.config/cliproxy/token")"

export ANTHROPIC_BASE_URL="http://127.0.0.1:8317"
export ANTHROPIC_AUTH_TOKEN="$PROXY_TOKEN"
unset ANTHROPIC_API_KEY

claude --model gpt-5.6-sol --effort xhigh
```

初回起動時にはワークスペースの信頼確認が表示される。

`~/YOUR_PROJECTS` 全体ではなく、対象プロジェクトのルートで起動する。

``` zsh
cd "$HOME/YOUR_PROJECTS/example-project"
claude --model gpt-5.6-sol --effort xhigh
```

Claude Code内で確認:

    /status
    /model
    /effort

別ターミナルでプロキシログを確認:

``` zsh
container logs -f cliproxy
```

モデル自身へモデル名を質問する方法は検証にならない。 Claude CodeのステータスとCLIProxyAPIのログを使う。

## claude-solラッパーを作成する

``` zsh
cat > "$HOME/.local/bin/claude-sol" <<'ZSH'
#!/bin/zsh
set -euo pipefail

TOKEN_FILE="$HOME/.config/cliproxy/token"
BASE_URL="http://127.0.0.1:8317"

if [[ ! -r "$TOKEN_FILE" ]]; then
  print -u2 "Missing token: $TOKEN_FILE"
  exit 1
fi

TOKEN="$(<"$TOKEN_FILE")"

if ! curl -fsS \
  --max-time 3 \
  "$BASE_URL/v1/models" \
  -H "Authorization: Bearer $TOKEN" \
  >/dev/null; then
  print -u2 "CLIProxyAPI is not reachable."
  exit 1
fi

  ANTHROPIC_BASE_URL="$BASE_URL" \
  ANTHROPIC_AUTH_TOKEN="$TOKEN" \
  ANTHROPIC_MODEL=gpt-5.6-sol[1m] \
  ANTHROPIC_SMALL_FAST_MODEL=gpt-5.6-luna[1m] \
  CLAUDE_CODE_AUTO_COMPACT_WINDOW=372000 \
  CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1 \
  CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK=1 \
  CLAUDE_CODE_SUBAGENT_MODEL=gpt-5.6-sol \
  CLAUDE_CODE_ALWAYS_ENABLE_EFFORT=1 \
  CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY=3 \
  CLAUDE_CODE_NO_FLICKER=1 \
  ENABLE_TOOL_SEARCH=false \
  exec claude --model gpt-5.6-sol "$@"
ZSH

chmod 700 "$HOME/.local/bin/claude-sol"
```

`~/.local/bin` がPATHにない場合:

``` zsh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> "$HOME/.zshrc"
source "$HOME/.zshrc"
```

利用方法:

``` zsh
cd "$HOME/YOUR_PROJECTS/example-project"
claude-sol
```

引数もそのまま渡せる。

``` zsh
claude-sol --continue
```

## コンテナ操作

一覧:

``` zsh
container list --all
```

停止:

``` zsh
container stop cliproxy
```

再開:

``` zsh
container start cliproxy
```

ログ:

``` zsh
container logs -f cliproxy
```

削除:

``` zsh
container delete --force cliproxy
```

Apple `container` のシステムを停止すると、CLIProxyAPI以外のコンテナも停止する。

``` zsh
container system stop
```

再起動後は必要なコンテナを個別に開始する。

``` zsh
container system start
container start cliproxy
```

## イメージ更新

``` zsh
IMAGE='docker.io/eceasy/cli-proxy-api:latest'

container image pull "$IMAGE"

container stop cliproxy 2>/dev/null || true
container delete --force cliproxy 2>/dev/null || true
```

その後、常駐起動用の `container run` を再実行する。

`latest` は更新によって互換性が崩れる可能性がある。 安定動作を確認した後は、利用可能な固定バージョンタグを使う方が安全。

## トラブルシュート

#### `path '.../config.yaml' is not a directory`

原因は単一ファイルを直接bind mountしていること。

使用しない:

``` zsh
--mount "type=bind,source=$HOME/.config/cliproxy/config.yaml,target=/CLIProxyAPI/config.yaml"
```

使用する:

``` zsh
--mount "type=bind,source=$HOME/.config/cliproxy,target=/config"
```

``` zsh
--config /config/config.yaml
```

#### `DNS: transient error` または `Connection refused`

確認:

``` zsh
container run --rm alpine:latest sh -lc '
  cat /etc/resolv.conf
  nslookup auth.openai.com
'
```

既定DNS `192.168.64.1` が拒否される場合、すべての対象コンテナへ外部DNSを指定する。

``` zsh
--dns 1.1.1.1
```

#### OAuth後に `ERR_CONNECTION_REFUSED`

ログインコンテナに次のポート公開が必要。

``` zsh
-p 127.0.0.1:1455:1455
```

さらにDNS指定が必要。

``` zsh
--dns 1.1.1.1
```

#### `Authentication failed`

コンテナからOpenAI認証サーバーを名前解決できるか確認する。

``` zsh
container run --rm \
  --dns 1.1.1.1 \
  alpine:latest \
  nslookup auth.openai.com
```

成功後にOAuthログインを再実行する。

#### `/v1/models` へ接続できない

``` zsh
container list --all
container logs cliproxy
```

CLIProxyAPI設定は次である必要がある。

``` yaml
host: "0.0.0.0"
port: 8317
```

ホスト側の公開設定:

``` zsh
-p 127.0.0.1:8317:8317
```

#### API認証エラー

``` zsh
PROXY_TOKEN="$(cat "$HOME/.config/cliproxy/token")"
```

リクエストヘッダー:

``` zsh
-H "Authorization: Bearer $PROXY_TOKEN"
```

`~/.config/cliproxy/token` と `config.yaml` の `api-keys` が一致していることを確認する。

## セキュリティ

8317番ポートはローカルホストだけへ公開する。

``` zsh
-p 127.0.0.1:8317:8317
```

次のように全インターフェースへ公開しない。

``` zsh
-p 8317:8317
```

保護対象:

    ~/.config/cliproxy/token
    ~/.config/cliproxy/config.yaml
    ~/.local/share/cliproxy/auth

推奨権限:

``` zsh
chmod 600 "$HOME/.config/cliproxy/token"
chmod 600 "$HOME/.config/cliproxy/config.yaml"
chmod 700 "$HOME/.local/share/cliproxy/auth"
```

※この構成はClaude Codeの公式な非Claudeモデル対応ではないため、Claude Code、CLIProxyAPI、またはCodex認証方式の更新で互換性が崩れる可能性がある。

