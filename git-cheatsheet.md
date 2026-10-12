# Git 速查卡

## 一眼看懂四个区域

```
工作区(你改的代码) --add--> 暂存区(购物车) --commit--> 本地仓库(存档) --push--> 远程(云端)
```

- `add` = 把改动放进购物车（挑哪些要一起存档）
- `commit` = 结账打包、盖时间戳（存在自己电脑上）
- `push` = 把包裹寄到云端（别的机器/别人能看到）
- `tag` = 给某个包裹贴便利贴「这是 baseline」

## 唯一固定的顺序

```
改代码 → commit → push → (tag 可选)
```

tag 是给**已经存在的提交**起名字，所以必然在 commit 之后。顺序敲错了 git 会直接报错，
不会把仓库搞坏。

## 日常只需要两条

```bash
git commit -am "说明"     # -a 帮你做了 add（只对已跟踪的文件有效）
git push
```

只有两种情况需要手敲 `git add`：

1. 新建的文件第一次进 git
2. 改了一堆东西，但只想提交其中一部分

## 想到什么敲什么

| 你想干的事 | 敲什么 |
| --- | --- |
| 改了已有文件，想存档 | `git commit -am "说明"` |
| 新文件第一次进 git | `git add 文件` → `git commit -m "说明"` |
| 存档并备份到云端 | `git commit -am "说明"` → `git push` |
| 先看要提交什么 | `git diff --cached` |
| 这版要当对比基准 | `git tag -a exp1 -m "lr=1e-3"` → `git push origin exp1` |
| 回到旧版看一眼 | `git switch --detach exp1`，看完立刻 `git switch main` |
| 只要某个旧版文件 | `git show exp1:src/model.py` |
| 撤销没提交的改动 | `git restore 文件` |
| 撤销已提交的改动（安全） | `git revert <提交号> --no-edit` |
| 不确定自己在哪 | `git status` |

## 迷路了怎么办

永远先敲 `git status`。它只回答两件事：**你在哪**、**有什么没提交**。

| 看到什么 | 怎么办 |
| --- | --- |
| `On branch main` | 正常，放心干活 |
| `HEAD detached at ...` / `HEAD (no branch)` | 先 `git switch main`，别在这里提交 |
| `fatal: You are not currently on a branch` | 同上 |
| `(END)` 或 `:` 卡住 | 按 `q` 退出分页器 |
| push 被拒，提示远程有新提交 | `git pull --rebase origin main` 再 `git push` |
| 想找回刚才丢掉的提交 | `git reflog` |

## 看历史

```bash
git log --oneline --graph --all      # 图形化看所有分支
git diff exp1 exp2                   # 两个版本之间差什么
git diff main origin/main            # 本地和远程差什么
git log -p -- src/                   # 只看某个目录改了什么
```

## 远程仓库（一次性配置）

```bash
git remote add origin git@github.com:hongwei1532/git-lab.git
git push -u origin main              # -u 记住上游，以后直接 git push
git fetch                            # 只下载远程最新，不动你的文件
git reset --hard origin/main         # 让云端覆盖本地（丢弃本地改动！）
git push --force-with-lease          # 让本地覆盖云端（重写公开历史，慎用）
```

SSH 走 443 端口（校园网封 22 时用），写进 `~/.ssh/config`：

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```

## 实验场景（跑模型专用）

- **开跑前**先提交 + 打标签，把"这个结果对应哪版代码"固定下来
- **提交信息里写配置和结果**：`git commit -am "exp: lr=1e-4 acc=0.883"`，
  这样 `git log --oneline` 就是实验记录本
- 想同时跑两个版本：`git worktree add ../gputest-old exp1`

`.gitignore` 必写：

```
.venv/
__pycache__/
data/
checkpoints/
runs/
outputs/
wandb/
*.pth  *.pt  *.safetensors  *.bin  *.log
```

权重、数据集、日志、输出目录**不要**进 git（每改一次存一整份，仓库会爆炸）；
但 `config.yaml`、seed、`pyproject.toml`/`requirements.txt` **必须**进，否则实验无法复现。

## 别做的事

- 别对**已推送**的历史用 `git reset --hard`（会让协作者的历史对不上），要用 `git revert`
- 别往协作仓库 `git push --force`（用 `--force-with-lease`，且只在自己的仓库里用）
- 别把 API key、token、私钥提交进 git（`~/.ssh/id_ed25519` 这个没有 `.pub` 的文件）
- 双盲投稿期间，论文代码仓库不要公开，用 anonymous.4open.science 生成匿名链接
