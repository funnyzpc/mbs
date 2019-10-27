### GO实现简单(命令行)工具:sftp,文檔压解,RDS备份,RDS备份下载

内容提要：

+ 1.远程连接linux执行sftp文件下载
+ 2.window下文件解压缩(tar、gz、zip)
+ 3.window下文件解压缩及带密码解压(zip)
+ 4.window下调用阿里雲RDS api查询备份并下载
+ 5.GO语言命令行工具技巧

```
      首先，写这篇博文的一个出发点是：我无法在window的cmd命令行下实现日期的加减(還有其他問題)，当然這不是没有实现的方法，而是实现起来很难维护难度较大，光插件都够我折腾了，另外window自带的powershell也可以实现，不过作为一个java渣来说真的难了点儿，因为又要熟悉powershell语法从零开始
      后来，我换了个思路，想用代码+第三方开源插件(依赖)实现以上功能；至于，目前我有对Python、java、js、Go、甚至Rust，这些都有或多或少的涉猎，分析了一遍，发现使用半静态或者脚本语言实现很easy，不过有一个问题：你每部署一台机器都要安装语言环境如Python、java,虽然可以跨平台，不过太臃肿了部署一个几兆的应用要安装一个几百兆的语言环境，实在太浪费了内存，js呢又太弱，需要自己造轮子，可以剔除，Rust速度快，不过编写的难度太大，很难考虑，
      最后我选用GO作为以上工具的语言，当然这里不得不说一下使用GO的好处：语法简单、跨平台、代码安全、静态打包：这个很重要，可以直接打windows下的可执行程序，也可以打linux可执行程序[交叉编译]，这样就可以在部署的时候不用动手又动脚的安装语言环境，配置环境变量之类的乱七八糟的东西，同时安全度也很复合我的需求，例如打成一个可执行包后就自带破解难度，更牛掰的是还可以使用upx对可执行包进行加壳，加壳有三个好处：几乎无法破解、可执行应用体积大大缩小(比我的一个应用打包后有16MB，加壳后只有3MB左右)、易于分发(当然这个是建立在加壳之上在)，一切准备就绪，这一篇我就简单的聊一聊我用GO如何实现这类Tools。

```

  由于本章只是個人實現的工具分享，因為這些工具有一定的灵活性,(若需要改造或實現)这时就需要您有一定的GO语言基础。


#### **1.远程连接** example: [log_bak01.go](https://github.com/funnyzpc/rds_backup/blob/master/service/log_bak01.go)

  这里的我主要用到 `github.com/pkg/sftp` 与 `golang.org/x/crypto/ssh`,一个是执行sftp命令，一个是建立ssh连接的，因为sftp是建立在安全的ssh连接上的
样例中有我实现实现linux日志拉取的功能的完整代码，，这里就不展示具体实现代码(参考样例)，就简单说说实现步骤吧：

+ 建立ssh配置
    - `config := &ssh.ClientConfig{...`
+ 建立建立ssh的TCP连接(ssh是建立在TCP连接上的)
    - `client, err := ssh.Dial("tcp", "服务器地址:端口", config)`
+ 使用ssh连接建立一个sftp连接J(sftp在使用完毕后必须close())
    - `sftp, err := sftp.NewClient(client)`
+ 打开一个Linux系统文件(在本地文件写入后远程文件必须close())
    - `srcFile, err := sftp.Open("/路径/文件01.log." + time + ".zip")`
+ 创建一个本地下载文件(本地文件写入完成后需要close())
    - `dstFile, err := os.Create(targetPath + "/文件01.log." + time + ".zip")`
+ 将链接的远程文件写入到本地下载文件
    - `srcFile.WriteTo(dstFile)`
+ 以上步骤的具体代码可以参考: [log_bak01.go](https://github.com/funnyzpc/rds_backup/blob/master/service/log_bak01.go)


#### **2.window下文件解压缩(tar、gz、zip)** example: [gzip_util.go，unzip_util.go](https://github.com/funnyzpc/rds_backup/tree/master/util)

  由于在解决实际问题的时候面临的问题比较复杂，光一个压缩包就有zip、tar、gz还有tar+gz的方式，具体使用的时候用到的依赖有
`archive/tar`、`compress/gzip`、`archive/zip`、`path/filepath`及GO自带的基本依赖等等...

+ 对于zip就比较简单
    - 首先你得傳入一個zip文件全路徑，然後使用zip的读模式open这个zip文件
        - `r, err := zip.OpenReader(fullZipFile)`
    - 遍历这个读取的zip文件，并循环(循環完畢需要將這個zip文件close())
        - `for _, f := range r.File {...`
    - 每循环到一个目录的时候在local创建这个文件夹
        - `os.MkdirAll(path, f.Mode())`
    - 每循环到一个文件的时候先在local创建目录并以写模式open这个文件，然后将循环到的文件写入到这个local文件(寫入完成需要close())
        - `os.MkdirAll(filepath.Dir(path), f.Mode())`
        - `f, err := os.OpenFile(path, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, f.Mode())`
        - `_, err = io.Copy(f, rc)`

+ 對於tar+gz文件的處理方法
    - 首先您需要以讀模式Open這個文件(在這個壓縮文檔全部解壓後需要close())
        - `fr, err := os.Open(srcFilePath)`
    - 因為大多數這種組合的壓縮文檔都是先tar然後再gz,所以這裏我們就先使用gzip依賴讀取這個文檔
        - `gr, err := gzip.NewReader(fr)`
    - 讀取gzip成功後，這時候需要使用tar依賴讀取這個tar文檔
        - `tr := tar.NewReader(gr)`
    - 遍歷循環這個讀取到的tar文檔並寫入目錄及文件(注意local文檔在寫入完成之後需要close())
        - `for {...`


#### **3.window下zip文件带密码解压** example: [unpzip_util.go](https://github.com/funnyzpc/rds_backup/blob/master/util/unpzip_util.go)

  其實官方給的example中並沒有帶秘密的解壓縮，這個問題困擾了我幾個小時，最終在我碰到有網友寫的這個依賴才得以解決:`github.com/yeka/zip"`,再次表示十分
感謝，在此能將example共享出來也算是功德一件哈～

  這裏的處理其實十分簡單,其實就是在每次循環zip文件的時候判斷一下`IsEncrypted()`,在true的時候`SetPassword(password)` ,後面使用io之後的文件就是
非加密文件了，so easy ～

+ 需要使用依賴的Open這個zip文件
    - `r, err := zip.OpenReader(fullZipFile)`
+ 遍歷循環這個zip文件
    - `for _, f := range r.File {...`
+ 在每循環到一個文件及目錄的時候設置一下password
    - `f.SetPassword(password)`
+ 將當前讀取到的文件及目錄寫入到local
    - `func writeFile(filePath string, f *zip.File) error {...`


#### **4.window下调用阿里雲RDS api查询备份并下载** example: [main1.go](https://github.com/funnyzpc/rds_backup/blob/master/main1.go)

  其實這是對前幾個功能對一個綜合，我對目的是下載阿里雲的RDS的物理備份並解壓,當然你需要先參閱官方api文檔，在這裏[DescribeBackups](https://help.aliyun.com/document_detail/26273.html?spm=a2c4g.11186623.6.1544.7bb06a5fq6e7IY)
其實一開始我並不知道,所有的備份文件都需要查詢到其下載連結後才可下載，獲取到下載連結後其他的事情就按部就班了～，以下是我實現的思路，可參閱～

+ 參閱官方文檔並使用api查詢備份文件(官方有api的調試代碼)
    - `client, err := rds.NewClientWithAccessKey("cn-hangzhou", "您的accessKeyId", "您的accessKeySecret")`
    - `request := rds.CreateDescribeBackupsRequest()`
    - `respJsonStr, err := client.DescribeBackups(request)`
+ 因為查詢到的數據為json的字符串形式，這時候需要將json轉換成struct
    - `data := &BackupInfo{}`
    - `json.Unmarshal(respJsonStr.GetHttpContentBytes(), data)`
+ 循環或直接調用這個結構體實例獲取到下載地址及相關參數
    - `downloadUrl := data.Items.BackupItem[0].BackupDownloadURL`
    - `backupDate := data.Items.BackupItem[0].BackupStartTime[0:10]`
+ 下載文件
    - `err := util.DownloadFile(downloadUrl, filePath)`
+ 解壓文件
    - `err2 := util.UnTarGz(filePath, unTarGzFilePath)`


#### **5.GO语言命令行工具技巧** example [main3.go](https://github.com/funnyzpc/rds_backup/blob/master/main3.go)

  一開始我是想將打包好的tools部署後使用命令行調用，這樣會顯得靈活一些，後來覺得這樣使用太過與零碎了，而且window下的執行環境也是個問題，最後才做決定將一組
功能當堵打包，然後使用windows的計畫任務調用，不過既然作為一種可行的方式(linux下比較可行)，所以就參閱了個簡單的Demo，讀者可以根據這個Demo改寫上述功能

  這裏是結合著命令行實現了個文件下載功能

+ GO語言開發環境自帶os包，使用os.Args獲取調用的所有調用的參數，這個`args\[0\]`是你打包好的exe可執行程序本身(windows環境下)
    - 比如你的命令行是 `main_exec.exe https://www.xxx.com/path/xx.zip D:/tmp`
    - 使用`os.Args`獲取到的參數有仨 os.Args\[0\]:`main_exec.exe`，os.Args\[1\]:`https://www.xxx.com/path/xx.zip`,os.Args\[2\]:`D:/tmp`
+ 從命令行參數中獲取下載地址和目錄參數
    - `url := os.Args[1]` `filename := os.Args[2]`
+ 調用download工具下載這個網絡文件
    - `err := util.DownloadFile(url, filename)`


#### **最後**

  本章的內容比較零散，望讀者諒解，另外，以上內容的所有代碼(包括已經打包好的exe程序)我已推送至github [rds_backup](https://github.com/funnyzpc/rds_backup)
這些代碼全部使用GO語言實現，當然以上內容可能並不完整，全黨是拋磚引玉，一個解決問題的方式，如有需要改進或優化的地方請留言哈～