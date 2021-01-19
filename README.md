# GitHubのPRで表示されるdiffについてテスト

## 要約
- GitHubの`ブランチB->ブランチA`のPRで表示される差分は`B`と`BとAのmerge-base`との差分
  - `B`と`A`の最新コミットとの差分 **ではない**
  - `git diff A..B`ではなく、`git diff A...B`の差分がGitHubのPRでは表示される

## 試してみる
### ブランチを2つ(A,B)用意して、それぞれで **おなじ** 変更を加える
```
A: --1--2
B:    \_3
(2,3の時点で同じ変更をする)
```

### B -> A のPRを出す。差分を確認
差分: https://github.com/takunoko/GitHub_diff/pull/1/files

=> AとBの最新コミットの比較なら差分が出ないはずだが差分が出ている。

### 差分
```
$ git diff branch-B branch-A
$ git diff branch-B..branch-A
$ git diff branch-B...branch-A
diff --git a/test.txt b/test.txt
index 7951742..520f3b8 100644
--- a/test.txt
+++ b/test.txt
@@ -1,4 +1,4 @@
-1:
+1: hoge
 2:
 3:
 4:
$
```
`git diff branch-B branch-A`と`git diff branch-B..branch-A`は同じ

## 異なる差分を加える
### ブランチCを作成して、異なる変更を同じ部分に加える
```
A: --1--2
B:    \_3
C:    \_4
(2,4は異なる変更)
```

### C -> A のPRを出す。差分を確認
差分: https://github.com/takunoko/GitHub_diff/pull/2/files

=> AとCの差分だが、C側の追加変更だけがあるように見える。(コンフリクトの解決に進もうとすると両方の差分が出てくる)

### 差分
```
$ git diff branch-C branch-A
diff --git a/test.txt b/test.txt
index 6de422f..520f3b8 100644
--- a/test.txt
+++ b/test.txt
@@ -1,4 +1,4 @@
-1: huga
+1: hoge
 2:
 3:
 4:
$ git diff branch-C...branch-A
diff --git a/test.txt b/test.txt
index 7951742..520f3b8 100644
--- a/test.txt
+++ b/test.txt
@@ -1,4 +1,4 @@
-1:
+1: hoge
 2:
 3:
 4:
```
GitHubのPRで表示されている差分は`git diff branch-C...branch-A`の方。

## まとめ
- GitHubのPRで表示される差分はmerge-baseからの差分
  - 全く同じ変更を加えた場合は変更があるが追加しただけに見える。コンフリクトしない。
  - 異なる変更を加えた場合は、片側にだけ変更があるようにPRでは表示される。コンフリクトする(それはそう)
  - ↑のどちらも差分は同じように見える。
- 自動PRなどで両方のブランチに同じ変更を加えたあとにPRを作っても差分が表示されるのはこのため。
