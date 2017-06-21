# 環境的準備\(for Node.js\)

#### 如果環境已經架設好，只是要佈署新的Node專案，可以直接從第11點開始。

1.從現有的映像檔裡建立一個Instance，區域看情況，台灣的話可以選asia-east1，速度可能會比較快。在下方【允許HTTP】及【允許HTTPS】的地方記得打勾，否則網頁會連不到。

> 這裡的範例是以Debian GNU/Linux 8 \(jessie\)的映像檔當範例，如果是Centos的話，有些命令會稍微不太一樣。

2.Instance建立好之後，點選【SSH】，可以直接用網頁的方式SSH進入該Instance。

3.下載Node.js並安裝：

```
sudo curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo ln -s /usr/bin/nodejs /usr/bin/node
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
```

10.設定pm2在系統重開機後自動啟動之前透過pm2 save所記錄的應用程式：

```
sudo pm2 startup
```

11.接下來就可以佈署程式，建立應用程式的目錄：

```
cd /home
sudo mkdir www
cd www
sudo mkdir helloworld
cd helloworld
```

12.初始化git，並將某個repo加入到git的remote：

```
sudo git init
sudo git remote add origin https://LoginName:Password@github.com/MedialandDev/2017_05_ML_GitHub_Manager.git
```

> LoginName是GitHub上的名稱，不是email\(例如:ML-Jason\)，Password是密碼

13.從GitHub上fetch，並強制讓local的檔案與GitHub上同步：

```
sudo git fetch origin
sudo git reset --hard origin/master
```

> 使用git reset --hard的原因是避免殘留檔案、衝突，一但reset --hard，就會強制的把local檔案與remote同步。

14.啟動應用程式，並save：

```
sudo npm i
sudo pm2 start index.js
sudo pm2 save
```

> 記得一定要sudo pm2 save，否則重新開機後，應用程式不會被pm2自動啟動
>
> 如果是Linux環境，可以考慮用pm2的cluster模式開啟多個instance執行，增加CPU的使用率。

15.回到執行個體列表，可以透過外部ip看看是否正常運作。

16.如果需要安裝GM\(graphicmagick\)：

```
sudo apt-get update
sudo apt-get install graphicsmagick
```

> 字型的話直接上傳到主機上，透過路徑的方式使用即可。

17.如果不想透過Git更新檔案，想直接用SFTP的話，記得修改檔案的Group：

```
cd /home
sudo chown -R :adm www
sudo chmod -R g+rwx www
sudo setfacl -Rdm g:adm:rwx ww
```

> 以上的命令是把www的目錄設為adm這個群組擁有，並且有rwx的權限
>
> 預設開發者都應該是adm，如果不是的話，自建群組並指派。



