# GitHubのPRで表示されるdiffについてテスト

## 要約
- GitHubの`ブランチB->ブランチA`のPRで表示される差分は`B`と`BとAのmerge-base`(共通の先祖)からの差分
  - `B`と`A`の最新コミットとの差分 **ではない**
  - `git diff A..B`ではなく、`git diff A...B`の差分がGitHubのPRでは表示

## 試してみる
### ブランチを2つ(A,B)用意して、それぞれで"おなじ"変更を加える
```
A: --1--2
B:    \_3
(2,3の時点で同じ変更)
```

### B -> A のPRを出す。差分を確認
PR差分: https://github.com/takunoko/GitHub_diff/pull/1/files
- [Aのtest.txt](https://github.com/takunoko/GitHub_diff/blob/branch-B/test.txt)
- [Bのtest.txt](https://github.com/takunoko/GitHub_diff/blob/branch-A/test.txt)

=> AとBの最新コミットの比較なら差分が出ないはずだが差分が出る

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
GitHubのPRで表示されている差分は`git diff branch-B...branch-A`の結果

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

### 参考
- [GitHubのプルリクエストの差分はどこと比較しているか？(git diffの&quot;..&quot;と&quot;...&quot;の違い) - Qiita](https://qiita.com/m-yamazaki/items/e57e357116e95ae370dc)
