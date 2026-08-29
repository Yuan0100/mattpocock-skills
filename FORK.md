# Fork 維運說明

這份文件只存在於 `mine` 分支，上游（`mattpocock/skills`）永遠不會有這個路徑，
所以 rebase 時不會產生衝突。

## 這個 fork 的結構

```
origin    → github.com/Yuan0100/mattpocock-skills.git   (fetch + push)
upstream  → github.com/mattpocock/skills.git            (只 fetch，push 已停用)
```

兩條分支各有明確職責：

| 分支   | 內容                                   | 推到哪裡      |
| ------ | -------------------------------------- | ------------- |
| `main` | 上游的**純鏡像**，零自有 commit        | `origin/main` |
| `mine` | 疊在 `main` 之上的個人客製化           | 目前只在本地  |

`main` 必須保持零自有 commit，這是整套流程的地基：只要它逐位元組等於
`upstream/main`，同步就永遠是 fast-forward，不會有衝突、不會有合併節點。
一旦在 `main` 上 commit 任何東西，`--ff-only` 就永久失效，往後每次同步都變成
要處理歷史分歧的麻煩事。**客製化一律進 `mine`。**

`remote.upstream.pushurl` 被刻意設成 `DISABLED`（一個不存在的位址），
就算誤打 `git push upstream` 也只會立刻失敗，推不到別人的 repo。

## 例行同步

```bash
# 1. 抓上游最新狀態（唯讀，隨時可跑）
git fetch upstream

# 2. 快轉 main
git checkout main
git merge --ff-only upstream/main

# 3. 同步 fork 的 main 回 GitHub
git push origin main

# 4. 把客製化重新疊到新的 main 上
git checkout mine
git rebase main
```

第 2 步的 `--ff-only` 是安全閥，意思是「不能快轉就直接失敗」。
它失敗代表 `main` 上混進了不該有的 commit，這時候該停下來查清楚，
不要改用一般 merge 硬合過去。

第 4 步選 rebase 而非 merge，是因為 `mine` 還沒推到任何 remote，
重寫 commit SHA 不影響別人；rebase 完客製化永遠疊在最新上游之上，
`git diff main..mine` 也永遠只顯示「我改了什麼」。
哪天 `mine` 推上遠端之後，就改用 `git merge --no-ff main`，
避免重寫已發佈的歷史。

## 檢查現況

```bash
# main 與上游的差距：輸出「upstream 獨有 / main 獨有」
git rev-list --left-right --count upstream/main...main   # 期望 0  0

# mine 帶了哪些客製化
git log --oneline main..mine
git diff --name-status main..mine
```

三個點是對稱差集，比 `git status` 的 ahead/behind 更適合比對沒有追蹤關係的分支。

## 目前的客製化

三個都是**修改上游既有檔案**，不是新增檔案：

- `skills/engineering/implement/SKILL.md`：改成走我的 git workflow（分支檢查、逐檔 add）
- `skills/engineering/triage/SKILL.md`：在本地 tracker 上提升 inbox 卡片
- `skills/engineering/setup-matt-pocock-skills/issue-tracker-local.md`：加上 inbox、resolved 狀態與封存

因為是修改而非新增，**上游改動同一份檔案時 rebase 會衝突**，這是預期內的，
不是設定壞掉。解法照 `/resolving-merge-conflicts` skill 走，
原則是保留上游的新內容，再把我的客製化重新套上去。

## 不要做的事

- 不要在 `main` 上 commit，會破壞 fast-forward 的前提
- 不要 `git push upstream`，雖然已被 `pushurl` 擋掉，但別依賴保險絲
- 不要把 fork 專屬內容寫進 `README.md` 或 `docs/`，那是上游的地盤，每次同步都要解衝突
