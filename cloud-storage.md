# Cloud Storage

Cloud Storage顧名思義就是用來存放檔案的地方，由於使用雲端時，我們有時可能會使用多台VM配合Load Balancer，除了程式的編寫需要是stateless以外，檔案的存放位置也是要注意的。

因此一但遇到【非靜態檔案】\(例如：網友上傳的圖片\)，我們不可能只將該檔案存放在某台VM上，這樣一來有些網友可能就會無法讀取該檔案，這時就需要將動態產生的檔案放置到Cloud Storage之上。

而且另外一方面，VM的磁碟空間有限，最佳的作法都是盡量避免動態產生的檔案直接存放在VM的磁碟裡。

> Google官方的Storage文件可參考：
>
> [https://cloud.google.com/storage/docs/how-to](https://cloud.google.com/storage/docs/how-to)

### 建立Storage

要建立一個Storage相對簡單，只要在Google主控台上點選【選單】-&gt; 【Storage】，進入Storage單元，再點選【建立Bucket】，給予一個名稱，就建立了一個新的Storage Bucket了。

未來如果有東西被上傳到這個Bucket裡，它的對外路徑就會是：[https://storage.googleapis.com/${bucketname}/${filename}](https://storage.googleapis.com/${bucketname}/${filename})

> 如果牽扯到跨網域讀取檔案的問題，例如：前端Canvas繪製跨網域的圖檔時會被"污染"，理論上可以配合Load Balancer將bucket納入某個domain之下。
>
> \*這一點尚未實作，待日後確定。

### 使用程式上傳檔案到Bucket

Google文件裡說明，如果App本身是在GCE上執行的話，是不需要任何認證就可以直接上傳的，但我們通常會在本機上先行開發，所以我們會需要取得Storage的【存取憑證】或是【API金鑰】，這樣才可以在本機端透過Google的Api來進行上傳。

我這邊只列出使用憑證進行的方式。其它方式請參考Google的文件。

#### 取得API憑證

1. 點選【選單】-&gt; 【API管理員】-&gt; 【憑證】。
2. 點擊【建立憑證】，選擇【服務帳戶金鑰】後，在服務帳戶裡可以選【Compute engine default service account】，然後下載json格式的憑證檔案。
3. 將json檔案放置到專案某個目錄下，之後程式會需要去讀取它。

#### Client Library \(For Node.js\)

> 官方文件請參考：
>
> [https://googlecloudplatform.github.io/google-cloud-node/\#/docs/storage/1.1.0/storage](https://googlecloudplatform.github.io/google-cloud-node/#/docs/storage/1.1.0/storage)

1.需要先安裝Google Storage Client Library

```
$ npm install --save @google-cloud/storage
```

2.在js檔裡引用並初始化

```js
const gcs = require('@google-cloud/storage')({
  projectId: 'grape-spaceship-123',
  keyFilename: '/path/to/keyfile.json'
});
```

> 這邊的projectId可在【資訊主頁】左上角找到。
>
> keyFilename則是剛剛上一步下載的json格式的API憑證。

3.設定上傳後的檔案是【公開的】

```js
const bucketname = 'your_bucket_name';
const bucket = gcs.bucket(bucketname);
bucket.acl.default.add({
  entity: 'allUsers',
  role: gcs.acl.READER_ROLE,
}, (err, aclObject) => {
  if (err) console.log(err);
});
```

> 注意：這一步驟很重要，否則上傳到Bucket上的檔案預設是無法被其它人讀取的。

4.將檔案上傳\(stream\)

```js
const filename = 'path/somepic.jpg'; //遠端的路徑
const writeStream = bucket.file(filename).createWriteStream({
  resumable: false,
  metadata: { contentType: 'image/jpeg' },
});
writeStream.on('error', (err) => {
  console.log(err);
});
writeStream.on('finish', () => {
  console.log('done');
});
writeStream.end(buffer.data);
```

> 注意：在上傳時createWriteStream裡帶的參數最好填寫metadata的contentType\(尤其是圖檔\)，否則Google有時會誤判，透過網址去query圖檔時會變成其它格式。

5.將檔案上傳\(file\)

    const filename = 'path/somepic.jpg'; //遠端的路徑
    const localfile = './xxxx.jpg'; //local端的路徑
    bucket
      .file(filename)
      .upload(localfile)
      .then(() => {
        console.log(`${localfile} uploaded to ${filename}.`);
      })
      .catch((err) => {
        console.error('ERROR:', err);
      });



