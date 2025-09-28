# Git 提交规范

## 1. 前言

Git 是一个免费开源的分布式版本控制系统，由 Linux 之父 Linus Torvalds 于 2005 年开发，最初的目的用于管理 Linux 内核的开发。因其高效的性能、便捷的分支管理、免费开源等优秀特性，一经推出，很快在全球范围内得到广泛使用，成为最流行的版本控制系统。

当我们使用 Git 对代码进行版本管理时，规范的提交信息（Commit Message）可以帮助团队更好地理解代码变更的历史记录，提高项目的可维护性和协作效率。

## 2. Commit Message 的作用

规范的 Commit Message 具有以下作用：

1. **提供更多的历史信息**：方便快速浏览代码变更历史
2. **便于查找关键信息**：可以过滤特定类型的提交
3. **自动生成 Change Log**：可以直接从 Commit 生成发布说明
4. **便于代码审查**：清晰的提交消息帮助审查者理解提交目的
5. **便于问题定位**：帮助快速定位和理解问题
6. **项目管理和追踪**：帮助项目管理者跟踪开发进度和问题修复情况
7. **提高项目整体质量**：促进团队形成良好的工程实践

## 3. Commit Message 规范

目前业界广泛采用的规范是 [Conventional Commits](https://www.conventionalcommits.org/)（约定式提交），它受到了 Angular 提交约定的启发。

### 3.1 基本结构

约定式提交规定每个提交消息由三个部分组成：Header（必需）、Body（可选）和 Footer（可选）。

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 3.2 Header

Header 部分又分为三个部分：类型（type）、范围（scope，可选）和描述（description）。

#### 类型（Type）

类型字段用于说明提交的类型，必须是以下类型之一：

- **feat**：新增功能（feature）
- **fix**：修复 Bug
- **docs**：文档变更（documentation）
- **style**：代码格式调整，不影响代码功能（空格、格式化、缺失分号等）
- **refactor**：代码重构，既不是新增功能，也不是修改 Bug
- **perf**：性能优化
- **test**：添加或修改测试代码
- **build**：构建系统或外部依赖项的变更
- **ci**：CI 配置文件和脚本的变更
- **chore**：其他不修改源代码或测试文件的变更
- **revert**：撤销之前的提交

#### 范围（Scope）

范围字段是可选的，用于说明本次提交影响的范围，如：

```
feat(api): 添加用户认证接口
```

范围可以是组件名、模块名、文件名等。

#### 描述（Description）

描述部分是对变更的简短描述：

- 使用第一人称现在时祈使句，如："change"而不是"changed"或"changes"
- 首字母不需要大写
- 结尾不需要句号

### 3.3 Body（可选）

Body 部分是对本次提交的详细描述，说明代码变更的动机，以及与以前行为的对比。注意：

- 使用第一人称现在时祈使句
- 应该说明代码变动的动机，以及与之前行为的对比

### 3.4 Footer（可选）

Footer 部分用于记录不兼容变动和关闭 Issue。

#### 不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以 `BREAKING CHANGE:` 开头，后面是对变动的描述、变动理由和迁移方法。

```
BREAKING CHANGE: 配置文件中的 `extends` 字段现在用于继承其他配置文件
```

也可以在 Header 的 type 后面添加 `!` 来标记破坏性变更：

```
feat(api)!: 重构用户认证接口
```

#### 关闭 Issue

如果当前提交针对某个 Issue，可以在 Footer 部分关闭这个 Issue：

```
Closes #123, #245, #992
```

## 4. 示例

### 4.1 简单的功能添加

```
feat: 添加用户登录功能
```

### 4.2 修复 Bug

```
fix: 修复用户无法登录的问题
```

### 4.3 带有范围的文档更新

```
docs(readme): 更新安装说明
```

### 4.4 包含详细说明的性能优化

```
perf: 优化列表渲染性能

将虚拟滚动应用于长列表，减少DOM节点数量，
显著提升了页面滚动的流畅度和内存占用。

Closes #234
```

### 4.5 包含破坏性变更的提交

```
feat(api)!: 重构用户认证API

BREAKING CHANGE: 用户认证API现在需要额外的安全令牌参数
```

## 5. 实践建议

1. **保持一致性**：团队内部应统一 Commit Message 规范
2. **一次提交做一件事**：每次提交应该有明确、单一的目的
3. **提交前检查**：确保提交的内容符合预期，避免将调试代码、临时文件等提交
4. **使用工具辅助**：可以使用 Commitizen、Commitlint 等工具辅助生成规范的提交信息
5. **定期回顾**：定期回顾提交历史，总结经验教训

## 6. 辅助工具

### 6.1 Commitizen

Commitizen 是一个撰写合格 Commit Message 的工具，它会引导开发者填写规范的提交信息。

安装：

```bash
npm install -g commitizen
```

使用 Angular 的提交规范：

```bash
commitizen init cz-conventional-changelog --save --save-exact
```

提交时使用 `git cz` 代替 `git commit`。

### 6.2 Commitlint

Commitlint 用于检查提交信息是否符合规范。

安装：

```bash
npm install -g @commitlint/cli @commitlint/config-conventional
```

配置：

```bash
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

### 6.3 Git Hooks

可以结合 Husky 使用 Git Hooks 来强制检查提交信息：

```bash
npm install husky --save-dev
npx husky install
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```

## 7. 总结

规范的 Git 提交信息不仅能提高代码库的可读性和可维护性，还能促进团队协作和项目管理。通过采用约定式提交规范并结合相关工具，我们可以有效提升开发效率和代码质量。

记住：好的提交信息应该清晰地表达"做了什么"和"为什么做"，而不仅仅是"怎么做"。 