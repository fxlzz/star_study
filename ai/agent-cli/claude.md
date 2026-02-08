[github: claude](https://github.com/anthropics/claude-code)

如果有 claude 的账号，那最好了。。。
需要有 google 账号和外国的电话

使用国内的模型 glm-4.7 也是不错的选择：

- 在`.claude/`目录下，创建`settings.json` 文件
```js
{

  "env": {
    "ANTHROPIC_AUTH_TOKEN": "77826001330e4faabe3f945fc9d03c88.q20mZZu5NEMmlIuc",
    "ANTHROPIC_BASE_URL": "https://open.bigmodel.cn/api/anthropic",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1
  }
}
```

- 改`.claude.json` 文件
	加一句：`"hasCompletedOnboarding": true` 跳过验证



