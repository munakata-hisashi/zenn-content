---
title: "GitHub Copilotを使ってTerminal上でコミットメッセージを生成する"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "github", "git", "copilot", "terminal"]
published: true
---

VSCodeのSource ControlではGitHub Copilotにコミットメッセージを書いてもらうことができます。普段gitの操作はターミナルで行うことが多いので、ターミナル上でもコミットメッセージを生成できるといいなと思いやり方を調べてみました。

## VSCode内のTerminalの場合

VSCode内のTerminalであれば、`git add`コマンドを実行した後にTerminalの左端のアイコンをクリックすると、「Generate Commit Message」というメニューが表示されます。ここをクリックするとGitHub Copilotがコミットメッセージを生成してくれます。

![](/images/vscode-terminal-commit-message.png)

pre-commitなどでコミット時に処理を行っているようなケースではSource ControlよりTerminalから実行すると便利かもしれないですね。

## VSCode以外のTerminalの場合

VScode以外の場合、GitHub CopilotのCLIが使えないかと調べてみました。`gh copilot suggest`というコマンドがあるのですが、これはあくまでコマンドの提案を行うもので、コミットメッセージのような文章を生成するものではありませんでした。

しかしsuggestコマンドへの入力には自然言語を使うことができるので、gitの差分を渡しつつコミットメッセージと共に`git commit`のコマンドを提案してもらうことができないかと考えました。

いくつかのプロンプトを試してみて、以下のようなgitのエイリアスを設定することで、`git cs`のようなコマンドでコミットメッセージを生成できるようにしてみました。

```zsh
cs = "!f() { diff=$(git diff --staged); gh copilot suggest -t shell \"Based on the following diff, generate a concise commit message and suggest a shell command that pipes this message into 'git commit -e -F -'. The message should follow the conventional commit format.\\n\\n${diff}\"; }; f"
```

このエイリアスを設定した後、`git add`してから`git cs`を実行すると、GitHub Copilotがコミットメッセージつきのコマンドを提案してくれます。

```zsh

% git cs

Welcome to GitHub Copilot in the CLI!
version 1.1.1 (2025-06-17)

I'm powered by AI, so surprises and mistakes are possible. Make sure to verify any generated code or suggestions, and share feedback so that we can learn and improve. For more information, see https://gh.io/gh-copilot-transparency

Suggestion:

  echo "docs: remove sample file section from README" | git commit -e -F -

? Select an option  [Use arrows to move, type to filter]
> Copy command to clipboard
  Explain command
  Execute command
  Revise command
  Rate response
  Exit
```

提案されたコマンドを実行すると、Copilotが生成したコミットメッセージが入力された状態でエディタが開きます。ここでメッセージを確認・修正してコミットすることができます。
日本語は生成できないのと、差分が大きい場合にどうなるかまでは試していませんが、簡単な変更であれば使えそうです。

