# 執行個體群體

執行個體群體是用來管理數個執行個體的Group，通常可以配合LoadBalancer來使用，讓流量可以分配到這個群體裡的每個執行個體。

執行個體群體有分成兩種：

#### 1. Managed Instance Group {#managed-instance-group}

這個群組裡的個體基本上都是由[執行個體範本](/執行個體範本)\(\*1\)所產生，所以每個個體在產生的時候都是一模一樣的。我們可以設定群組裡instance的數目來進行scaling，群組會自動產生或移除instance來符合需求。

這種群組裡的個體【可以支援】autoscaling、rolling updating\(\*2\)。

由於Managed Instance Group裡的個體都是動態產生，所以每個個體的程式編寫方式必需是【stateless】的，檔案系統也最好使用額外的storage處理。

> \*1. 執行個體範本，我們可以從現有某個instance的硬碟建立一個映像檔，然後產生一個template，之後新的VM就可以直接使用這個template產生一樣的實體。
>
> \*2. 如果有新的instance template，可以用rolling update來更新群組裡的所有個體。

#### 2. Unmanaged Instance Group {#unmanaged-instance-group}

照字面上的意思來看，在這裡面的執行個體基本上都是單獨獨立的個體，我們可以在裡面自由的增加、移除個體。

這種群組裡的個體【無法】autoscaling、rolling updateing。

