# Code Conventions

## 前言

這份開發守則的頁數不多，名字也不怎樣，但它確是實實在在有力量的

怎麼說呢？因為任何違反此本開發守則進行開發的工程師，一律都會被Fire掉

沒錯，霸道，就是它的力量來源

因為我們希望，工程師進來之後，對於程式可以快速上手，離開時也可以快速交接，一統Code Style是一種最簡單的手法

儘管不遵守就開除看似霸道，但我們也是講王道的，只要你有更好的想法，歡迎你提出你的想法，我們會放進守則中，但前題是你的想法需合乎以下三個準則

1. 簡單易記
2. 快速上手與交接
3. 減化操作步驟，達成既定功用

我們完全不希望開發守則變得太厚，最後厚得像奧德賽還是摩訶婆羅多這類的經典名著一樣，這樣誰可以記得起來且快速將它內化呢？

我們只歡迎可以減少這份開發守則字數，又可以兼顧Code Style的人來改變它

話說軟體工程師分為三個級別

* 上者，不用開發就能幫公司賺錢
* 中者，努力開發幫公司賺錢
* 下者，努力開發還讓公司賠錢

## 命名規則

一、凡任何常數，命名一律採用全部大寫
```
define(‘DATABASE’, ‘mysql’);
```

二、凡任何Class，命名一律首字第一字大寫，且需標記`Controller/Model`
```
class Login_Controller extends Controller
```

三、凡任何變數名與Function命名，一律全部小寫

三、請用_取代駝峰標記法
```
public function get_user_by_id
```

四、變數命名需清楚明白，要讓人可以一樣看出，故一律禁用簡寫

五、資料庫無論是表格名還是欄位名，除禁用簡寫與禁用駝峰標記外，且需要都寫上註解

六、凡使用`if..else`, `for`, `foreach`, `while`等，不管是不是一行解決，一律加上`{}`，但於view端請一律採用`<?php if(xxx):><?php endif;?>`

### 小技巧

一、於邏輯判斷時，請將一定有值的放在最前面，再與其他變數進行比較，好比
```
if(0 === count($rows))
if(‘’ === $user)
```

一、善用`@`與`(int)`等方式，可以少用到`isset()`、`is_numeric()`，好比
```
$user = (string) trim(@$_POST[‘user’]); //若$_POST[‘user’]不存在，加上@會變成空字串
$id = (int)(@$_POST[‘id’]); //若$_POST[‘id’]為字串，會自動變成0
```

二、善用`in_array()`進行判斷，好比
```
$accept = array(‘127.0,0.1’, ‘192.168.0.1’);
if(!in_array($myip, $accept)) {
    header(‘location:/login/deny_remote’);
}
```

三、盡可能使用cookies，因為session會吃主機資源，安全性的部份可透過token的方式補足，好比
```
public function get_token($message = ‘’) { 
    $limit_date = date(‘Y-m-d H’);
    $token = (string) trim($message);
    $token = md5(sha1($message . $limit_date));
    return $token
}
.
.
.
setcookies(‘user’, $this->get_token($username));
.
.
.
if($username === $this->get_token($username))
```

四、請用`if..elseif...else`取代`switch`，雖然`switch`比較快些，但`if`能做的功能比較多，也比較簡捷

五、可以透過一個`if`做掉的，就別透過`if...else`還是`if`裡面又包`if`，好比
```
if(!$query)
if(0 === count($query->fetchAll()))
```
可改成
```
if(!$query || 0 === $query->fetchAll())
```

六、嚴禁盡可能少做一些會增加迴圈loading的事情，好比
```
for($i = 0; $i < count($rows); $i++)
可改成
$count = count($rows);
for($i = 0; $i < $count; $i++)
```

## Controller與Model

一、凡`Controller/Model`，任一Function資料進來一律先用變數去接，型態一律要明白，好比
```
$id = (int) trim(@$_POST['id']);
$content = (string) trim(@$_POST['content']);
$user = (string) trim(@$_SESSION[‘user’]);
$passwd = (string) trim(@$_COOKIES[‘passwd’]);
```

二、變數接完之後，一律先判斷帶進來的參數是否合理，且要判斷到資料型態
```
if($id === (int) $xxxx) or if($content = (string) $xxx)
```

三、model下SQL，變數的部份一律用`bind`的方式，好比
```
$sql = 'SELECT * FROM guestbook WHERE id = :id';
$bind = array(':id', $id);
$query = $this->db->query($sql, $bind);
```

四、凡資料庫回傳之資料，一律檢查是否為true、false，是否有值等，好比
```
if(!$query || 0 === count($query->fetchAll()))
```

五、凡資料庫回傳之資料要供前台使用，一律用變數去接，好比
```
$view['id'] = $query[0]['id'];
$view['content'] = $query[0]['content'];
```

六、凡程式/資料錯誤回報與例外判斷，一律統一採用Try...Catch，好比
```
try{
    if($id === (int) $xxxx) or if($content = (string) $xxx) {
        throw new Exception(訊息/陣列/物件);
    }
} catch(Exception $e) {
    echo $e->getMessage(); //Or any process
}
```

七、凡Controller/Model，沒直接要讓網址呼叫還是直接使用，一律採用`private`

八、凡用`private`標示之Function，首字第一個字一律用_出頭，好比
```
private function _get_user_by_member
```

九、凡需提供給AJAX/CURL呼叫之Action，請一律採用`ajax_還是curl_`出頭，好比
```
public function ajax_login
public function curl_get_user
```

十、所有程式開發之前，先檢查一下有沒有function是做相同功能的，有現成的就用現成的

十一、常用到的程式可寫入到一個新增的Model或是Framework底層的Controller.php/Model.php供大家使用

## View

一、任何沒用到之js、css一律拿掉

二、css與`<style></style>`一律放到`<head></head>`之間

三、所有js程式，不管是從外面載入的，還是寫在View裡面的，一律放在`</body>`之前

四、為了不想html的id, name, class與css相互干擾，請採用以下命名方式
```
<input type=”text” data-id=”user” class=”xxx” value=”” />
<input type=”password” data-id=”password” class=”xxx” value=”” />
<button type=”button” role=”login” data-href=”/user/ajax_login”>送出</button>
```
名詞解釋
name | description
-----|------------
data-id | 相當於dom的id/name，但可多筆名字重覆
data-href | 相當於ajax時要呼叫的路徑
data-* | *可換成任何名字放其他參數用
role | 相當於觸發之後要進行的行為

五、凡任何AJAX，請一律採用以下結構進行開發，一律需有url, type, dataType, success, error
```
var user = $(‘input[‘data_id=user’]’).val();
var password = $(‘input[data_id=password]’).val();
var url = $(‘button[role=login]’).attr(‘data-href’);
.
.
.
$.ajax({
    url:url, //盡可能採用相對路徑
    type: ‘post’, //有機密資料，好比password的，請用post，其餘好比討論區的頁數就可用get
    dataType: ‘json’ //請一律採用json格式
    data: { //代入參數參考
        user:user,
        password:password
    },
    success: function(response) { //呼叫成功，callback接的變數請一律將名字改成response
    },
    error: function(xhr) { //呼叫失敗，callback接的變數請一律將名字改成xhr
    }
});
```

六、若開發上時程較趕，且無特殊要求，可將表單驗証透過後端php程式處理

七、凡用於view端使用之變數，請先判斷是否有用於html/js/css上之需求，針對需求進行不同判斷
```
<?php echo strip_tags(@$data[‘subject’]); //文章主旨?>
<?php echo htmlspecialchar(@$data[‘password’]); //使用者密碼?>
<?php echo @$data[‘content’]; //文章內文?>
```

八、凡ajax回傳之變數，請先判斷是否有用於html/js/css上之需求，針對需求進行不同判斷
```
$(‘input[data-id=subject]’).text(response.subject); //輸出之內容會一律變成文字
$(‘input[‘data-id=content’]’).html(response.content); //輸出之內容可以變成html tag
```

九、盡可能別在view做任何邏輯判斷，如果要做，請多用`<if(xxx):?><?endif;?>`取代`if{}`，也可以擅用短判斷

