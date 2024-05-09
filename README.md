# Giriş Doğrulamanın Atlatılması ve Kısıtlamaların Aşılması ( Solution of Bypass Restriction and validation of input ) Çözümü

Bu saldırıyı [anlatım sayfası](https://github.com/yasir723/giris-dogrulamanin-atlatilmasi-ve-kisitlamalarin-asilmasi)nda gördüğümüz gibi hackerler olarak incele kısmında bazı HTML elementlerin özelliklerini `Örneğin: disabled veya required` kaldırarak veya doğrudan `Ağ` kısmından boş veri göndererek veritabanı boş veya gereksiz bilgileriyle doldurabiliriz. Bu sayfada hackerler olarak değil geliştirici olarak bu saldırıya karşı sistemimizi nasıl korabileceğimizi öğreneceğiz:


Bu tür bir saldırıyı, anlatım sayfasında gördüğümüz gibi, hackerler olarak inceleme aracını kullanarak bazı HTML öğelerinin özelliklerini, örneğin disabled veya required, kaldırarak veya doğrudan Ağ sekmesinden boş veri göndererek gerçekleştirebiliriz. Böylece, veritabanını gereksiz veya boş bilgilerle doldurabiliriz. Ancak, bu sayfada, hackerler olarak değil, sistemin geliştiricileri olarak, bu tür saldırılara karşı sistemimizi nasıl koruyabileceğimizi öğreneceğiz.

Örnek olarak kullandığımız web sitede JavaScript kodunu kullanarak şifrenin en az küçük bir harfin içerdiğini ve en az 6 uzunlukta olması gerektiğini kıstlama yapılmış, biz de `disabled` özelliğini kaldırarak bu kıstlamayı atlatabildik, bu tür kısıtlamaları hackerin atlatmaması için istemci tarafında değil sunucu tarafında yazılması gerek. PHP kodunda yazarak hackerin ona erişemeyeceği için atlatması çok zor olacaktır. Ayrıca boş gönderme işlemi sadece istemci tarafında required özelliğini kullanarak kısıtlama yapmak yanlış olur gördüğümüz gibi onu aşabildik. Güvenliği sağlamak için girilen bilgileri sunucu tarafında boş olup olmadığını da kontrol etmemiz gerek.

Kullandığımız örnek web sitesinde, JavaScript koduyla şifrenin en az küçük bir harf içermesi ve en az 6 karakter uzunluğunda olması gerektiği gibi kısıtlamalar yapılmıştı. Ancak, biz `disabled` özelliğini kaldırarak bu kısıtlamayı atlatabildik. Bu tür kısıtlamaların, bir hackerin bunları atlatamaması için istemci tarafında değil, sunucu tarafında uygulanması gerekmektedir. PHP koduyla yazılan kısıtlamalar, kullanıcının erişimine kapalı olduğu için atlatılması çok daha zor olacaktır. Ayrıca, sadece istemci tarafında `required` özelliğini kullanarak boş gönderimleri engellemek için yeterli değildir, çünkü bunu aşabildiğimizi gördük. Güvenliği sağlamak için, girilen bilgilerin boş olup olmadığını kontrol etmek için sunucu tarafında da doğrulama yapmamız gerekmektedir.

Yeni bir kullanıcı eklemek için kullanılan PHP kodu:

```php
<?php
  //Database Authentication
  require("DBInfo.php");
  
  $userName = $_POST['userName'];
  $password = $_POST['password'];
  
  $connect = mysqli_connect($hostDB, $userDB, $passwordDB, $databaseDB);
  if (mysqli_connect_errno()) {
      die(" cannot connect to database " . mysqli_connect_error());
  }
  
  $query = "Insert into login(userName,password) VALUES ('" .
      $userName . "','" . $password . "')";
  
  $result = mysqli_query($connect, $query);
  if (!$result) {
      die(' Error cannot run query');
  }
  
  mysqli_close($connect);
?>
```

Bu kodda, kullanıcı adı (userName) ve şifre (password) bilgilerini okuduktan hemen sonra, bu bilgilerin boş olup olmadığını veya belirlediğimiz kısıtlamalara uyup uymadığını kontrol etmek için if komutlarını kullanmamız gerekiyor.


Eklenmesi gereken ve kontrol işlemini gerçekleştiren kod parçacığı:


```php
$errors = [];
if (empty($userName) || empty($password))
    $errors[] = "Kullanıcı adı ve şifre alanları doldurulmalıdır.";
if (!preg_match('/[A-Za-z]/', $password))
    $errors[] = "Şifre en az küçük bir harf içermelidir.";
if (strlen($password) <= 5)
    $errors[] = "Şifre en az 6 karakter uzunluğunda olmalıdır.";

if (count($errors)) {
    print_r($errors);
    exit();
}
```

Kodu güncelledikten sonra bu şekilde olup:

```php
<?php
  //Database Authentication
  require("DBInfo.php");

  $userName = $_POST['userName'];
  $password = $_POST['password'];

  $errors = [];
  if (empty($userName) || empty($password))
      $errors[] = "Kullanıcı adı ve şifre alanları doldurulmalıdır.";
  if (!preg_match('/[A-Za-z]/', $password))
      $errors[] = "Şifre en az küçük bir harf içermelidir.";
  if (strlen($password) <= 5)
      $errors[] = "Şifre en az 6 karakter uzunluğunda olmalıdır.";

  if (count($errors)) {
      print_r($errors);
      exit();
  }

  $connect = mysqli_connect($hostDB, $userDB, $passwordDB, $databaseDB);
  if (mysqli_connect_errno()) {
      die(" cannot connect to database " . mysqli_connect_error());
  }

  $query = "Insert into login(userName,password) VALUES ('" .
      $userName . "','" . $password . "')";

  $result = mysqli_query($connect, $query);
  if (!$result) {
      die(' Error cannot run query');
  }

  mysqli_close($connect);
?>
```

Yeni bir kullanıcı eklemeye çalışırken boş veya belirli kısıtlamalara uymayan veriler girildiğinde işlem gerçekleştirilmeyecektir.


`Disabled ve required` özelliklerini kaldırdıktan sonra yeni bir kullanıcı eklemeye çalıştığımızda, örneğin:

   
<div style="text-align: center;">
    <div style="margin: 0 auto; width: 80%;">
        <span style="margin-right: 15px;">Kullanıcı Adı: 123; Şifre: 123 </span>
        <span style="margin-left: 15px;">&&&& Kullanıcı Adı ve Şifre boş girilmiş</span>
    </div>
    <img src='https://github.com/yasir723/giris-dogrulamanin-atlatilmasi-ve-kisitlamalarin-asilmasi-cozumu/assets/111686779/a2d7825a-fce2-46f8-bcf9-451b30d6d997' style="width:49%;">
    <img src='https://github.com/yasir723/giris-dogrulamanin-atlatilmasi-ve-kisitlamalarin-asilmasi-cozumu/assets/111686779/80dc7534-0e77-4647-8723-8cfe56a629c4' style="width:49%;">
</div>


Hataları daha güzel bir şekilde gösterebilmesi için bu şekilde PHP kodunu yazabiliriz:


```php
$errors = [];
if (empty($userName) || empty($password))
    $errors[] = "Kullanıcı adı ve şifre alanları doldurulmalıdır.";
if (!preg_match('/[A-Za-z]/', $password))
    $errors[] = "Şifre en az küçük bir harf içermelidir.";
if (strlen($password) <= 5)
    $errors[] = "Şifre en az 6 karakter uzunluğunda olmalıdır.";

if (count($errors)) {

    echo "<div style='width:49%; margin: 0 auto;border-radius: 15px;padding: 15px;background: #e19a9a;' >Hata:<ul>";
    foreach ($errors as $eleman) {
        echo "<li>$eleman</li>";
    }
    echo "</ul></div>";

    exit();
}
```

<div align="center">
    <h2>Kullancı Adı ve Şifre Boş Girilmiştir</h2>
    <img src="https://github.com/yasir723/giris-dogrulamanin-atlatilmasi-ve-kisitlamalarin-asilmasi-cozumu/assets/111686779/8a90127b-3a3b-48d3-a9d0-8c793164164a">
</div>




