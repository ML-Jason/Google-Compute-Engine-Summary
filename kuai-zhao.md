# 快照

快照就是把目前正在執行的VM個體完整的copy一份起來。

通常快照的用法就是用來快速的複製一個VM個體出來，但【快照無法用來建立執行個體範本】。如果要建立範本，請參考[執行個體範本](/執行個體範本)的製作方式。

> 由於製作執行個體範本時需要將原本的VM停止並刪除，如果萬一需要原VM個體持續運作的話，我們可以用快照的方式將VM個體快照起來，再產生一個新的暫時VM個體，用這個新的個體去產生範本。
>
> 這樣一來原本的VM個體就不會被迫停止。\(但是步驟會多好幾步就是了\)


