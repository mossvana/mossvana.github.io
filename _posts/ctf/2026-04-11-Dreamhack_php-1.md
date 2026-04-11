---
title: "[Dreamhack] php-1 문제 풀이"
date: 2026-04-11
categories: [ctf]
---

### php-1
![image](/assets/images/ctf/php-1-1.png){: .align-center}
LFI란 Local File Inclusion의 약자로 서버 내부에 위치한 파일을 포함시켜 읽어오는 공격을 말한다<br>

![image](/assets/images/ctf/php-1-2.png){: .align-center}
![image](/assets/images/ctf/php-1-3.png){: .align-center}
List 페이지에 들어갔더니 flag.php와 hello.json이 보인다<br>

![image](/assets/images/ctf/php-1-4.png){: .align-center}
flag.php 로 들어가면 나오는 화면이다. 권한이 없어서 안보이는거 같다 
url을 `/?page=view&file=/var/www/uploads/flag.php` 이렇게 바꿔도 권한이 없다고 뜬다<br>

![image](/assets/images/ctf/php-1-5.png){: .align-center}
hello.json 으로 들어가면 나오는 화면이다<br>

![image](/assets/images/ctf/php-1-6.png){: .align-center}
View 페이지이다. 해당 코드는 아래와 같다

```php
<h2>View</h2>
<pre><?php
    $file = $_GET['file']?$_GET['file']:'';
    if(preg_match('/flag|:/i', $file)){
        exit('Permission denied');
    }
    echo file_get_contents($file);
?>
</pre>
```
“flag”를 막는것을 확인할 수 있다<br>

list.php 코드도 확인해보자

```php
<h2>List</h2>
<?php
    $directory = '../uploads/';
    $scanned_directory = array_diff(scandir($directory), array('..', '.', 'index.html'));
    foreach ($scanned_directory as $key => $value) {
        echo "<li><a href='/?page=view&file={$directory}{$value}'>".$value."</a></li><br/>";
    }
?>
```

![image](/assets/images/ctf/php-1-7.png){: .align-center}
url 인코딩을 해보았는데 먹히지 않았다 

php url 우회에 관한 정보를 구글링해보니 php wrapper에 대해 알게되었다

php는 파일 시스템 함수에서 사용할 수 있는 다양한 url 스타일 프로토콜에 대한 내장 wrapper를 제공한다고 한다 

- **php://filter/** - I/O stream을 다루는데 사용하며 base64 encoding으로 문서 열람
    
    예시: www.[websiteaddress].index.php?page=php://filter/convert.base64-encode/resource=/etc/passwd)
    

php://filter/convert.base64-encode/resource=/var/www/uploads/flag

![image](/assets/images/ctf/php-1-8.png){: .align-center}

이런식으로 떠서 개발자 도구에서 확인했다 
![image](/assets/images/ctf/php-1-9.png){: .align-center}
![image](/assets/images/ctf/php-1-10.png){: .align-center}

플래그: DH{bb9db1f303cacf0f3c91e0abca1221ff}
