# Laravel Frontend

## **Laravel Mix 安裝**
Laravel Mix為一個建立在webpack上層的配置區，其提供簡潔的API，協助使用者更輕鬆的定義資源工作任務。

### 執行安裝
預設Laravel專案建立後，內含的package.json就已經定義好了laravel mix這個依賴，<br/>
執行npm(yarn) install後就會安裝Laravel Mix。

> 註：當專案建立在windows分割區時，於Linux下執行npm install就會發生錯誤。<br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> 因為預設npm install會建立符號連結，而這在windows下並不被支援，所以會發生錯誤。<br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> 此時，須額外加上參數--no-bin-links，宣告不執行符號連結的建立。<br/>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> npm(yarn) install --no-bin-links<br/>
>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> (作者的示範是將專案建立在Linux分割區，再使用samba分享給windows。所以，並未出現類似的錯誤)

### 執行Laravel Mix任務
```
npm run dev

npm run production (亦會執行資源最小化)
```

### 監控資源修改
```
npm run watch

npm run watch watch-poll (某些環境下，可能出現資源檔案修改後，webpack仍無法更新，此時須加上watch-poll參數)
```

## **Laravel Mix Tasks**
### CSS編譯任務
Mix API：**less、sass、stylus**
```
mix.mixAPI( {原始資源檔路徑},{編譯儲存路徑} )
   .options({
       額外參數設定
   })

Ex.  mix.sass('resources/assets/sass/app.sass', 'public/css')
        .sass('resources/assets/sass/admin.sass', 'public/css');
```
PS. 編譯儲存路徑可以使用包含檔名的完整路徑，用以重新定義編譯後的資源檔案名稱

### CSS合併任務
Mix API：**styles**
```
Ex.
mix.styles([
    'public/css/vendor/normalize.css',
    'public/css/vendor/videojs.css'
], 'public/css/all.css');
```

### Javascript合併任務
Mix API：**scripts、babel**
```
Ex.
mix.scripts([
    'public/js/admin.js',
    'public/js/dashboard.js'
], 'public/js/all.js');
```
PS. babel的作用與scripts類似，但會額外將ES2015轉換回原生的javascript

### Javascript 整合式腳本任務
預設就提供了ECMAScript 2015編譯、模塊綁定、壓縮及合併，無須額外進行配置。<br/>
Mix API：**js**
```
Ex. mix.js('resources/assets/js/app.js', 'public/js');
```

### 第三方依賴提取任務
當應用程式本身的JS需要頻繁更新時，與其他第三方綑綁成一個大型JS檔，其並非一個好方式。<br/>
容易造成用戶端需頻繁重載JS資源檔，此時應該將第三方依賴提取出來獨立成一支vendor.js<br/>
Mix API：**extract**
```
mix.js('resources/assets/js/app.js', 'public/js')
   .extract(['vue','jquery'])
```
```
資源載請依下列順序(下列三支檔案即extract後的結果)
<script src="/js/manifest.js"></script>
<script src="/js/vendor.js"></script>
<script src="/js/app.js"></script>
```
PS. 關於這部分，起初並不是很能理解。覺得開始就別綁在一起不就得了? 為何要先綁後提呢?<br/>
      後來想想，或許是為了方便開發者設定吧!! 因為只需針對單一的app.js做依賴載入，<br/>
      之後一個簡單的extract就可自動分割，不用大費周章的修改webpack.mix.js的設定。<br/>
      對於沒有特殊需求的開發者，這種方式其實蠻方便的。

### 檔案、目錄複製任務
常用在將node_modules下特定資源重新定位到專案public下<br/>
Mix API：**copy、copyDirectory**
```
mix.copy('node_modules/foo/bar.css', 'public/css/bar.css'); // 複製目錄時，其下子目錄會被扁平化
mix.copyDirectory('assets/img', 'public/img'); // 不執行扁平化，保持既有的目錄結構
```
PS. 此API在run prod後並不會有最小化的效果，若需minify還是要透過合併或編譯的任務API

### 資源版控任務
Mix API：**version**
```
mix.js('resources/assets/js/app.js', 'public/js')
   .version();

版本化後的文件名稱會被執行hash，所以載入需改用mix函數，如下
<link rel="stylesheet" href="{{ mix('/css/app.css') }}">
```
```
// 僅於運行期間進行，開發階段不做版控
mix.js('resources/assets/js/app.js', 'public/js');

if (mix.inProduction()) {
    mix.version();
}
```

### 監控特定文件變化
自動將文件變化效果同步到瀏覽器，無須手動刷新(對PHP腳本亦同)<br/>
Mix API：**browserSync**
```
mix.browserSync('my-domain.dev');

或者

// https://browsersync.io/docs/options
mix.browserSync({
    proxy: 'my-domain.dev'
});
```
[更多的browserSync設定](https://browsersync.io/docs/options)

PS. 需執行npm(yarn) run watch後，才會自動刷新<br/>

### 關閉Laravel Mix通知任務
預設，Mix會發送通知每個編譯的結果。可透過API進行關閉。<br/>
Mix API：**disableNotifications**
```
mix.disableNotifications();
```

### 關閉css圖檔路徑重寫機制
預設，CSS編譯後，url()內部的圖檔相對路徑將會被改寫。模式如下：<br/>
以../images/example.png為例，先找到圖檔example.png，然後複製到 public/images 目錄下，<br/>
接著改寫url()內部的路徑為絕對路徑
```
編譯前
.example {
    background: url('../images/example.png');
}

編譯後
.example {
  background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
}
```
關閉此一機制方式如下：
```
mix.sass('resources/assets/app/app.scss', 'public/css')
   .options({
      processCssUrls: false
   });
```

### 資源映射任務
主要功用，提供開發者工具更多的偵錯資料，預設為禁用，可透過API開啟。<br/>
Mix API：**sourceMaps**
```
mix.js('resources/assets/js/app.js', 'public/js')
   .sourceMaps();
```

### webpack配置方式
Laravel Mix本身即含有預設的webpack配置，置放於node_modules/laravel-mix/setup/webpack.config.js內，<br/>
欲進一步異動配置，可使用下列兩種方法：
 * 使用Mix API **webpackConfig**覆寫預設<br/>
 (該API可包含任何原始的[webpack設定項](https://webpack.js.org/configuration/))
 ```
 Ex.
 mix.webpackConfig({
    resolve: {
        modules: [
            path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
        ]
    }
});
 ```
 * 完全自行配置
   * 先將node_modules/laravel-mix/setup/webpack.config.js複製到專案根目錄
   * 修改package.json內--config參數的指向
   * 於mix版本更新時，須手動將異動合併到專案根目錄下的webpack.config.js

## **其他Laravel Mix特性**
### 存取.env內部設定
 * 藉由使用前綴MIX_來宣告mix設定，如 MIX_SENTRY_DSN_PUBLIC=http://example.com
 * 在Mix環境下，使用process.env來存取設定值，如 process.env.MIX_SENTRY_DSN_PUBLIC<br/>
    (注意!! 在watch階段更改MIX設定值時，須重啟watch任務才能生效)