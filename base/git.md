# Git 规范

## 提交规则

- 不要自动提交代码，除非明确要求
- 提交前确保代码能正常运行
- 直接在 main/master 上提交，保持敏捷

## 提交 Message 格式

```
<type>(<scope>): <subject>
```

冒号后有空格。type 枚举：

| type | 用途 |
|------|------|
| feat | 新增功能 |
| fix | 修复 bug |
| docs | 文档注释 |
| style | 代码格式（不影响运行） |
| refactor | 重构优化（非新功能、非修复） |
| perf | 性能优化 |
| test | 增加测试 |
| chore | 构建过程或辅助工具变动 |

超过两个要点时用列表：

```
feat(web): implement email verification workflow

- Add email verification token generation service
- Create verification email template with dynamic links
- Add API endpoint for token validation
```
