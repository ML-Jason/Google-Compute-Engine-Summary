# 環境的準備\(for Node.js\)

1.從現有的映像檔裡建立一個Instance，區域看情況，台灣的話可以選asia-east1，速度可能會比較快。在下方【允許HTTP】及【允許HTTPS】的地方記得打勾，否則網頁會連不到。

> 映像檔通常都會選擇Debian GNU/Linux 8 \(jessie\)

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

15.回到執行個體列表，可以透過外部ip看看是否正常運作。


