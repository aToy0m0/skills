# skills

このリポジトリは、Claude Code / OpenAI Codex で使うための Agent Skills を置くためのものです。

## Skills

- [`aws-cdk-cloudformation-design`](./aws-cdk-cloudformation-design/)
- [`aws-cli-operations`](./aws-cli-operations/)
- [`aws-cost-audit`](./aws-cost-audit/)
- [`cognitive-rhythm-writing`](./cognitive-rhythm-writing/)（[出典](./cognitive-rhythm-writing/SOURCE.md)）
- [`de-ai-ui`](./de-ai-ui/)
- [`generate-docs`](./generate-docs/)
- [`gcloud-cli-operations`](./gcloud-cli-operations/)
- [`git-daily-operations`](./git-daily-operations/)
- [`internal-doc-authoring`](./internal-doc-authoring/)
- [`japanese-tech-writing`](./japanese-tech-writing/)（[出典](./japanese-tech-writing/SOURCE.md)）
- [`pleasanter-script-deploy`](./pleasanter-script-deploy/)
- [`prepare-public-release-docs`](./prepare-public-release-docs/)
- [`system-deploy-check`](./system-deploy-check/)
- [`system-design`](./system-design/)

## Codex での設定

- ユーザー単位: `$HOME/.agents/skills/<skill-name>/SKILL.md`
- プロジェクト単位: `<repo>/.agents/skills/<skill-name>/SKILL.md`
- 反映されない場合: Codex を再起動

## Claude Code での設定

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

