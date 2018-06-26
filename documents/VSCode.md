# Visual Studio Code Installation Steps
編輯器主要安裝環境為Windows，所以相關指令執行部分需在提示命令列下執行


## Visual Studio Code 相關環境安裝
#### 安裝VSCode
自行到官網https://code.visualstudio.com/下載安裝包並執行

注意：安裝時，僅需在提示Choosing the default editor used by Git時改選用<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;
Use VisualStudioCode as Git's default editor即可，其餘步驟皆使用預設

#### 安裝Git
自行到官網https://git-scm.com/下載安裝包並執行

#### 提示命令列下確認Git安裝
```
git --version
```

#### 設定Git使用者資訊
```
git config --global user.name {User Name}
git config --global user.email {User Email}
```

#### 檢查Git全域設定值
```
git config --list
```

#### 安裝TortoisGit (Optional)
官網下載套件直接安裝即可。<br/>
因為作者習慣使用，所以安裝此軟體並透過VSCode的外掛來間接使用。<br/>
當然VSCode本身及一些外掛也有提供Git專案的控制，所以這段全憑個人喜好，不見得要Fllow。

#### 安裝wamp (Experimental)
官網下載直接安裝即可。<br/>
此部分是因為需要PHP的執行檔，剛好作者電腦有安裝wamp，所幸就直接用了。<br/>
有什麼問題，屆時遇到再行調整囉!!<br/><br/>
老話一句，這段全憑個人喜好，不見得要Fllow。


#### 安裝Composer
官網下載套件直接安裝即可。<br/>
此部分是因為部分外掛是Base在Composer套件，所以Windows下也需要Composer環境，<br/>
以便安裝相關套件

## Visual Studio Code 相關外掛安裝
樣式、圖案及快捷設定完全是個人喜好，所以大可忽略。<br/>
必裝套件則是作者集網路大成與自身喜好所整理出來的，僅供參考。

 - Slack Theme
 - file-icons
 - Notepad++ keymap
 - VSCode Plugins文件下的必裝套件<br/>
   phpcs需要原始套件中的執行檔，為了順利生效，需在Windows下用composer裝phpcs<br/>
   phpmd需要原始套件中的執行檔，為了順利生效，需在Windows下用composer裝phpmd<br/>
   SSHExtension需先設定好ftp-sample的server設定才能在終端機進行遠端伺服器連接<br/>
   Get Require Plugin List From [VSCodePlugins.txt](https://github.com/Internaltide/Laradep/blob/master/documents/VSCodePlugins.txt)

## Visual Studio Code 相關設定
經由 檔案->喜好設定->設定 進入組態設定畫面<br/>
將欲修改的設定項目從左側複製到右側，並將值修改為自訂值，ctrl+s儲存後立即生效。<br/>
以下為作者的設定範例：(註1)
```
{
    // 樣式(兩個設定值皆需安裝對應的套件才有用)
    "workbench.colorTheme": "Slack Theme",
    "workbench.iconTheme": "file-icons-colourless",

    // File Save Hook
    "files.trimFinalNewlines": true,
    "files.trimTrailingWhitespace": true,

    // 搜尋模式
    "search.smartCase": true,

    // 自動儲存
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 300000,

    // 整合式終端機
    "terminal.integrated.cursorBlinking": true,
    "terminal.integrated.fontSize": 16,
    "terminal.integrated.fontFamily": "monospace",
    //"terminal.integrated.shell.windows": "wsl.exe",

    // 編輯器設定
    "window.closeWhenEmpty": true,
    "editor.minimap.enabled": false,
    "editor.fontSize": 16,
    "editor.tabSize": 4,
    "editor.fontLigatures": true,
    "editor.fontFamily": "Fira Code",
    "editor.cursorBlinking": "smooth",
    "editor.quickSuggestions": {
        "other": true,
        "comments": true,
        "strings": true
    },
    "editor.mouseWheelZoom": true,
    "editor.scrollBeyondLastLine": true,

    // PHP
    "php.validate.executablePath": "C:/wamp64/bin/php/php7.1.16/php.exe",

    // PHP IntelliSense
    "php.executablePath": "C:/wamp64/bin/php/php7.1.16/php.exe",
    "php.memoryLimit": "1G",

    // phpcs
    "phpcs.executablePath": "C:/Users/Darrenting/AppData/Roaming/Composer/vendor/bin/phpcs.bat",
    "phpcs.standard": "PSR2",

    // PHPMD
    "phpmd.command": "C:/Users/Darrenting/AppData/Roaming/Composer/vendor/bin/phpmd.bat",

    // PHP DocBlocker
    "php-docblocker.author": {
        "name": "Author NAme",
        "email": "Author Email"
    },
    "php-docblocker.extra": [],

    // TODO Highlight
    "todohighlight.isCaseSensitive": false,
    "todohighlight.defaultStyle": {"backgroundColor": "lightblue"},
    "todohighlight.keywords": ["WARNING","KLUDGE","TRICKY"],

    // SSHExtension
    "sshextension.openProjectCatalog": true,

     // PHP Debug
    "debug.allowBreakpointsEverywhere": true,
    "debug.inlineValues": true,

    // New Configure
}
```

##  Visual Studio Code工作區建立
因為許多套件指令都被設定成只能在工作區環境下使用，<br/>
故建立專案所屬工作區，方便開發。<br/><br/>
#### 建立專案磁碟區，連線到Samba分享出來的專案根目錄
 - 打開建立網路磁碟區的介面
 - 介面上的資料夾欄位請輸入 \\{Samba Host IP}\projects<br/>
   如 \\192.168.1.124\projects
 - 往後，我的電腦下面就會多出一個磁碟區，那就是你在Linux下的專案目錄
#### 建立工作區
 - 於Visual Studio Code，以開啟資料夾的模式打開專案資料磁碟
 - 另存工作區，於專案磁碟區的根目錄下建立工作作區檔案
 - 往後只需以VSCODE打開工作區檔案，即可進入專案工作區
#### 工作區隱藏目錄
工作區的根目錄會自動建立隱藏目錄.vscode，放置相關設定檔如下
 - launch.json  => 儲存Xdebug的設定
 - setting.json => 用以設定工作區的IDE設定，會覆寫使用者組態設定<br/>
   ...

## Visual Studio Code Debug設定
#### Debug設定初始化
 - 專案工作區環境下切換到Debug頁面下
 - 頁面左上角處，請切換至預設的Listen for XDebug並啟動偵錯模式
 - 此時會自動開啟launch.json的設定檔，請於設定檔尾端疊加以下設定<br/>
   {
        "name": "Listen for XDebug of Laradep Host",
        "type": "php",
        "request": "launch",
        "pathMappings": {
            "/var/www/{專案名稱1}": "${workspaceRoot}\\{專案名稱1}",
            "/var/www/{專案名稱2}": "${workspaceRoot}\\{專案名稱2}"
        },
        "port": 9000,
        "log": true
    }

#### Debug運行方式
 - 開啟VSCODE並進入專案工作區
 - 於工作區模式下開啟偵錯模式
 - 開啟遇偵錯的檔案，設定中斷點
 - 瀏覽器開啟對應的URL，並附加上?XDEBUG_SESSION_START={IDE KEY}以啟動Debug Session
 - VSCode會開始進入偵錯執行階段(註2)(註3)

### 註解
1. 組態面板上，左側為預設值，右側為使用者自訂
2. Laradock有提供xdebug開關指令，於Laradock目錄下執行<br/>
    ./php-fpm/xdebug start      開啟<br/>
    ./php-fpm/xdebug stop       關閉<br/>
    ./php-fpm/xdebug status     查詢狀態，看PHP運行是否夾帶Xdebug<br/>
3. 作者發現偶爾XDebug的client、server連線會莫名的無法建立，必須關閉容器、所屬VM、網頁以及VSCODE<br/>
    然後全部重開，才能正常連線。<br/>
    (此問題尚待測試及解決)



<br/><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laravel development environment documents
------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Document Home&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/README.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux OS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Samba&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Docker&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Docker.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laradock&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VSCode