# 執行個體範本

執行個體範本\(Instance Template\)可以用來建立新的Instance，通常建立的範本都是從原本的某個Instance複製而來的。

如果我們需要一個Managed Instance Group，讓Group裡面的個體可以autoscaling的話，就一定需要使用執行個體範本。

#### 建立範本的方法

1. 首先建立一個Instance，該Instance的來源可以是從映像檔、快照或是其它磁碟、甚至是另外一個範本。
2. 在Instance裡做了一些修改後\(安裝程式、修改設定檔... etc\)，就可以準備將它製作成範本。
3. 在該Instance的設定裡，將【刪除執行個體時一併刪除開機磁碟】這個選項取消。
4. 刪除Instance，僅留下它的開機磁碟。
5. 到【映像檔】，上方點選【建立映像檔】，【來源磁碟】的欄位選取剛剛留下來的磁碟。
6. 映像檔建立時，可以選填【系列】，這樣可以規劃映像檔的版本，之後方便更新執行個體範本。
7. 建立完映像檔後，點選【執行個體範本】-&gt;【建立執行個體範本】，選取剛剛建立的映像檔。


