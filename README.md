# skills

このリポジトリは、Claude Code / OpenAI Codex で使うための Agent Skills を置くためのものです。

## Codex での設定（超シンプル）

- ユーザー単位: `$CODEX_HOME/skills/<skill-name>/SKILL.md`（macOS/Linux の既定: `~/.codex/skills`）
- プロジェクト単位: `<repo>/.codex/skills/<skill-name>/SKILL.md`
- 反映: Codex を再起動

## Claude Code での設定（超シンプル）

- プロジェクト単位: `<repo>/.claude/skills/<skill-name>/SKILL.md`
- ユーザー単位: Claude Code plugin として `skills/<skill-name>/SKILL.md` を同梱し、`/plugin install` でインストール

## 公式ドキュメント

- Codex Agent Skills: https://developers.openai.com/codex/skills/
- Claude Code Skills: https://docs.claude.com/en/docs/claude-code/skills
- Claude Code Plugins（skills をユーザー単位で配布する場合）: https://docs.claude.com/en/docs/claude-code/plugins

## Links

- https://github.com/agent/skills

## License

MIT

