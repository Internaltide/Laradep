# Laravel Localization Feature

## **Laravel Localization 目錄結構**
```
-resources
  -lang
    -en
      messages.php
    -zh-tw
      messages.php
    -{語系目錄}
      {語系檔案}
    ......
    ...
    .
```

## **Localization control function via App Facade**
 * setLocale($locale)
```
App::setLocale('en');  // 設定為英文語系
```
 * getLocale()
 ```
App::getLocale();  // 取得目前語系
 ```
 * isLocale($locale)
```
App::isLocale('en');  // 判斷是否為英文語系
```

## **Relational config in config file  app.php**
```
'fallback_locale' => 'en',  //定義語系之翻譯字串不存在時，要回到哪個語系去找(此例為回去英語系檔找該翻譯字串ˋ)
```

## **翻譯字串定義方法**
### 以短字串為鍵
翻譯檔格式：**php**<br/>
翻譯檔位置：**resources/lang/{語系目錄}/{語系檔}.php**
 ```
 <?php

// resources/lang/en/messages.php

return [
    'welcome' => 'Welcome to domain management'
];
 ```

### 以翻譯字串為鍵
翻譯檔格式：**json**<br/>
翻譯檔位置：**resources/lang/{語系}.json**
 ```
 // resources/lang/es.json

 {
    "I love programming.": "Me encanta programar."
}
 ```

 ## **翻譯值取得**
 * PHP Program: 使用輔助函數__()<br/>
 ```
 echo __('messages.welcome');  // 使用{檔名}.{短鍵名}的串接字串作為參數，取得welcome的翻譯

echo __('I love programming.');  // 使用長鍵名作為參數，取得I love programming的翻譯
 ```
 * Blade Template
 ```
{{ __('messages.welcome') }}  // 直接使用語法{{ }}去包裹輔助函數

@lang('messages.welcome')  // 使用directive指令@lang()
 ```

 ## **翻譯字串占位符**
 ### 占位符以冒號: 作為開頭予以宣告
```
'welcome' => 'Welcome, :name',

echo __('messages.welcome', ['name' => 'dayle']); // 利用輔助函數的第二參數定義要取代占位符的值
```
&nbsp;>>> 上述例子將返回Welcome, dayle

### 占位符的特有性質，大小寫同步
```
'welcome' => 'Welcome, :NAME', // 如果占位符皆大寫，則翻譯值皆大寫。 Ex. Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // 如果占位符僅首字母大寫，則翻譯值首字母大寫。 Ex. Goodbye, Dayle
```

 ## **翻譯字串複數規則**
 ### 使用pipe character區分單複數
 ```
 'apples' => 'There is one apple|There are many apples',
 ```

 ### 使用數字範圍指定翻譯字串，並使用trans_choice(索引鍵,計數值)取得對應的翻譯
 ```
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

echo trans_choice('messages.apples', 10);
```
&nbsp;>>> 上述例子將返回There are some

### 同時使用占位符與複數規則的翻譯字串
```
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 6]);
 ```
 &nbsp;>>> 上述例子將返回6 minutes ago

 ## **複寫套件的翻譯字串**
 有些套件會有自己的翻譯檔，直接更改套件核心並不是一件值得倡導的事情。<br/>
 Laravel提供覆寫套件翻譯檔的方法，使用方法如下：
 * 於resources/lang下建立套件專有的目錄，並將覆寫翻譯檔置於其下。<br/>
    **resources/lang/vendor/{package}/{locale}**<br/>

**Example: 欲覆寫套件skyrim/hearthfire英語系翻譯檔messages.php下的特定翻譯字串**<br/>
Solve:<br/>
 * 建立目錄 resources/lang/vendor/hearthfire/en
 * 建立覆寫翻譯檔messages.php，並置放在前一步建立的目錄之下
 * 覆寫翻譯檔只需放要覆寫的翻譯字串設定。其餘的仍會回去讀取套件的翻譯值