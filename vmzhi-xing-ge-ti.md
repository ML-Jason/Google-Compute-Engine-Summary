## VM執行個體

Google Compute Engine基本上就是VM的服務，我們可以藉由幾種方式建立VM：

#### 1. Google提供的映像檔

Goolge預先提供了許多的映像檔可以讓我們直接產生VM，其中包括了各種版本的作業系統或MsSQL。

#### 2. 自定映像檔

我們也可以自己產生自己的映像檔，透過產生最基本的VM，在裡面進行設定的修改、安裝程式之後，保留VM的開機磁碟，使用開機磁區產生新的映像檔。

如果要自訂[執行個體範本](/執行個體範本)通常也需要先建立自己的映像檔。

映像檔可以進行版本的管理。

#### 3. 快照

很像映像檔，但如果要產生映像檔，需要將VM關閉，快照不用。

> 快照無法用來建立自訂的執行個體範本。

#### 4. 現有磁碟

每個VM產生時，會有自己的開機磁碟，我們可以使用某個VM的開機磁碟用來產生另外一個VM。跟映像檔也很像，差別只在於映像檔是基於現有磁碟產生的，多了一個步驟，且映像檔可以進行版本的管理。

> 現有磁碟也無法用來建立自訂的執行個體範本。


