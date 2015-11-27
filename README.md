# Ansible group_vars precedure

ansibleのgroup_varsって優先順位ってどうなってんのよ?

cliオプションで渡すやつとか、role defaults含めた全体としての優先順位はこんなの。

http://docs.ansible.com/ansible/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable

しかし、group(group_vars)間についてはとくに記述がない(多分)。

## 知りたかったこと

 * 複数のgroupに同一hostが属してるとどうなるのか
 * groupに階層がある場合
 * 各group間での変数の参照はできるのか

### 検証環境

というわけで、適当なplaybookとinventoryファイルを用意して検証してみた。


```console
brew install ansible
git clone https://github.com/lamanotrama/ansible-vars-procedure.git
cd ansible-vars-procedure
./play
```

で実行できる。

## わかったこと

 * 単一のホストをinventoryで複数のgroupに属することは可能
 * 複数に属してallで実行しても 実行されるのは一回のみ
   * これはlocalコネクションで実行してるせいかも
 * group_varsは全groupのものが見える
 * group_varsで同じ値を定義した場合の優先順位は
   * 子group > 親group > それ以外のgroup > all
 * 「それ以外のgroup」内での優先順位がよくわからない。辞書順でもなさそう
   * inventoryファイルの見た目の定義順は関係なさそう
   * 組み込みのgroup_names配列には辞書順でならんでいるが…
 * 優先順位に関係なく、各grouo_vars間でそれぞれに定義されている変数が参照可能
   * group階層としては親であるgrupから、子のgroupのvarを参照できる(変数のvalue側で展開できる)

大体こんな感じ。尚、このprojectの内容そのままで実行だけでなく、inventoryの内容や順序も変更して実行してわかったことも含む。

### こうしよう

これらを踏まえておもったこと

 * 親子関係のない複数groupに属するような設計はできるだけ避けよう
 * どうしてもそうなる場合でも、むやみに変数増やすのはやめよう
 * 子の変数を親が参照できるのは直感に反してキモいので、できるだけやらんとこう

