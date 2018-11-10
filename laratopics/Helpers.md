# Laravel 輔助方法

> ~~~
> Laravel 包含了各式各樣的全域「輔助」PHP 函數，而大部分的函數都有被框架自己來使用。如果你也覺得他們很方便
> 的話，可以在應用程式中隨意的使用它們。
> ~~~

## 可用方法
### 陣列與物件
|                |               |             |           |            |            |
| ---------- |:---------- |:-------- |:------- |:------- |:-------- |
| array_add | array_collapse | array_divide | array_dot | array_except | array_first |
| array_flatten | array_forget | array_get | array_has | array_last | array_only |
| array_pluck | array_prepend | array_pull | array_random | array_set | array_sort |
| array_sort_recursive | array_where | array_wrap | data_fill | data_get | data_set |
| head | la |||||

### 路徑
|                |               |             |           |            |            |
| ---------- |:---------- |:-------- |:------- |:------- |:-------- |
| app_path | base_path | config_path | database_path | mix | public_path |
| resource_path | storage_path |||||

### 字串
|                |               |             |           |            |            |
| ---------- |:---------- |:-------- |:------- |:------- |:-------- |
| __ | camel_case | class_basename | e | ends_with | kebab_case |
| preg_replace_array | snake_case | starts_with | str_after | str_before | str_contains |
| str_finish | str_is | str_limit | Str::orderedUuid | str_plural | str_random |
| str_replace_array | str_replace_first | str_replace_last | str_singular | str_slug | str_start |
| studly_case | title_case | trans | trans_choice | Str::uuid ||

### URL
|                |               |             |           |            |            |
| ---------- |:---------- |:-------- |:------- |:------- |:-------- |
| action | asset | secure_asset | route | secure_url | url |

### 其它
|                |               |             |           |            |            |
| ---------- |:---------- |:-------- |:------- |:------- |:-------- |
| abort | abort_if | abort_unless | app | auth | back |
| bcrypt | blank | broadcast | cache | class_uses_recursive | collect |
| config | cookie | csrf_field | csrf_token | dd | decrypt |
| dispatch | dispatch_now | dump | encrypt | env | event |
| factory | filled | info | logger | method_field | now |
| old | optional | policy | redirect | report | request |
| rescue | resolve | response | retry | session | tap |
| today | throw_if | throw_unless | trait_uses_recursive | transform | validator |
| value | view | with ||||