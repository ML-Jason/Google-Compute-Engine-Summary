# 環境的準備\(for Node.js\)

#### 如果環境已經架設好，只是要佈署新的Node專案，可以直接從第11點開始。

1.從現有的映像檔裡建立一個Instance，區域看情況，台灣的話可以選asia-east1，速度可能會比較快。在下方【允許HTTP】及【允許HTTPS】的地方記得打勾，否則網頁會連不到。

> 這裡的範例是以Debian GNU/Linux 8 \(jessie\)的映像檔當範例，如果是Centos的話，有些命令會稍微不太一樣。

2.Instance建立好之後，點選【SSH】，可以直接用網頁的方式SSH進入該Instance。

3.下載Node.js並安裝：

```
sudo curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

上述如果是要安裝node 7的版本，請把第一行改成

```
sudo curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
```

4.安裝Git \(更新code的時候使用Git比較方便\)：

```
sudo apt-get install -y git
```

5.安裝nginx：

```
sudo apt-get install -y nginx
```

6.修改nginx的設定\(進入nginx設定目錄，將原本的default備份，再建立新的\)：

```
cd /etc/nginx/sites-available
sudo mv default default.bak
sudo vi default
```

7.新建的default請參考下方

```
server {
  listen 80;

  location / {
    proxy_pass http://127.0.0.1:3001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_cache_bypass $http_upgrade;
  }
}
```

如果設定好SSL，需要強制把HTTP導向到HTTPS的話，會像是這樣：

```
server {
  listen 80;
  listen [::]:80;                                                  
  listen 443 ssl;

  server_name www.medialand.com.tw;

  ssl_certificate /etc/nginx/ssl/pizzahutevent07.crt;                                                                                                
  ssl_certificate_key /etc/nginx/ssl/pizzahutevent07.KEY;    

  if ($http_x_forwarded_proto = "http") {
    return 301 https://$host$request_uri;
  }

  location / {                                                                                  
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;                                                      
  }                                                                                             
}
```

8.啟動nginx：

```
sudo service nginx restart
```

9.安裝pm2

```
sudo npm install -g pm2
sudo pm2 startup
```

10.建立執行pm2的使用者，移除它的密碼\(這是為了讓pm2的log存放在可存取的目錄\)，並給予sudo與adm的權限\(GCP預設的管理者都是adm群組\)

```
sudo adduser webuser
sudo passwd -d webuser
sudo usermod -aG sudo webuser
sudo usermod -aG adm webuser
```

> 之後就統一使用webuser進行pm2的操作，如果要切換成webuser，就輸入：
>
> su webuser
>
> 如果要跳離webuser，就輸入：
>
> exit

11.設定pm2在系統重開機後自動啟動，並安裝logrotate：

```
su webuser
pm2 startup
pm2 install pm2-logrotate
```

> 注意!!! sudo pm2 startup只會發生在用sudo啟動pm2的情況，如果不是sudo執行pm2的話，會需要改成：
>
> sudo env PATH=$PATH:/usr/bin /usr/lib/node\_modules/pm2/bin/pm2 startup systemd -u webuser --hp /home/webuser
>
> \(其中webuser是執行的人，其實只要執行pm2 startup，系統就會提省要輸入什麼指令\)
>
> pm2的log預設是放在啟動的使用這home目錄之下，假設啟動者是user，則pm2的log會放在：
>
> /home/user/.pm2/logs/
>
> 因此，如果需要取得pm2的log檔，請盡量避免使用sudo啟動app，否則pm2的log存放的地方只有root才有權限存取。
>
> 關於pm2的log我還沒有摸熟，可參考官網資料：
>
> [http://pm2.keymetrics.io/docs/usage/log-management/](http://pm2.keymetrics.io/docs/usage/log-management/)

12.接下來就可以佈署程式，建立應用程式的目錄：

切換成webuser，建立目錄。

```
su webuser
cd /home
sudo mkdir www
```

安裝acl\(要用來設定整個目錄的預設權限\)，並設定www目錄裡所有新增的檔案都預設為adm群組，這樣其它adm群組的成員才有辦法修改檔案。

```
sudo apt-get install acl
sudo setfacl -Rm d:g:adm:rwX,g:adm:rwX ./www
```

設定好之後，就可以在www裡新增檔案或目錄。

```
cd www
mkdir helloworld
cd helloworld
```

13.初始化git，並將某個repo加入到git的remote：

```
git init
git remote add origin https://LoginName:Password@github.com/MedialandDev/2017_05_ML_GitHub_Manager.git
```

> LoginName是GitHub上的名稱，不是email\(例如:ML-Jason\)，Password是密碼

14.從GitHub上fetch，並強制讓local的檔案與GitHub上同步：

```
git fetch origin
git reset --hard origin/master
```

> 使用git reset --hard的原因是避免殘留檔案、衝突，一但reset --hard，就會強制的把local檔案與remote同步。

15.啟動應用程式，並save：

```
npm i
pm2 start index.js
pm2 save
```

> 記得一定要pm2 save，否則重新開機後，應用程式不會被pm2自動啟動。
>
> 如果需要取得pm2的log，請不要用sudo執行pm2。

16.回到執行個體列表，可以透過外部ip看看是否正常運作。

17.如果需要安裝GM\(graphicmagick\)：

```
sudo apt-get update
sudo apt-get install graphicsmagick
```

> 字型的話直接上傳到主機上，透過路徑的方式使用即可。



