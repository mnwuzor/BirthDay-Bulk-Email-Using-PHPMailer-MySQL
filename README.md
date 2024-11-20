# BirthDay-Bulk-Email-Using-PHPMailer-MySQL
This is my own revised version of *PHPMailer/examples /mailing_list*  @ https://github.com/PHPMailer/PHPMailer/blob/master/examples/mailing_list.phps

## My version is a mass email sent as birthday message

```
*Here I set the currrent day and current month from where the birth day and birth month will be triggered*
<?php
$currentDate = date('Y-m-d'); //this will get the current date
echo $birthMonth = date('F'); //this will get the current month
echo $birthDay = date('d'); //this will get the current day
?>

<?php
*//Import the PHPMailer class into the global namespace*
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

error_reporting(E_STRICT | E_ALL);

*Here I set my time zone*
date_default_timezone_set('Africa/Lagos');

*Here I comment out the PHPMailer Composer Autoload so I can use mine*
//require '../vendor/autoload.php';

require ('PHPMailer-master/src/PHPMailer.php');
require ('PHPMailer-master/src/Exception.php');
require ('PHPMailer-master/src/SMTP.php');

//Passing `true` enables PHPMailer exceptions
$mail = new PHPMailer(true);

//Create a custom birthday card or page and save it as contents.html. If you choose to change the name to something else, remember to change the name here too
//You can use this link to create a dummy birthday card https://www.codewithrandom.com/2023/12/30/happy-birthday-wishes-html-and-css/
$body = file_get_contents('contents.html');

$mail->isSMTP();
$mail->Host = 'mail.mydomainname.com'; //here add your SMTP host name
$mail->SMTPAuth = true;
$mail->SMTPKeepAlive = true; //SMTP connection will not close after each email sent, reduces SMTP overhead
$mail->Port = 25;
$mail->Username = 'info@mydomainname.com';
$mail->Password = 'youremailpassword';
$mail->setFrom('info@mydomainname.com', 'Birthday Wish');
$mail->addReplyTo('info@mydomainname.com', 'Birthday Wish');
$mail->Subject = 'Birthday mailing list test';

//Same body for all messages, so set this before the sending loop
//If you generate a different body for each recipient (e.g. you're using a templating system),
//set it inside the loop
$mail->msgHTML($body);
//msgHTML also sets AltBody, but if you want a custom one, set it afterwards
$mail->AltBody = 'To view the message, please use an HTML compatible email viewer!';

//Connect to the database and select the recipients from the mailing list or table that have not yet been sent to
//You'll need to alter this to match your database
$mysql = mysqli_connect('localhost', 'your_db_username', 'your_db_password');
mysqli_select_db($mysql, 'your_db_name');

$result = mysqli_query($mysql, "SELECT email, fullname FROM tablename WHERE bday='$birthDay' AND bmonth='$birthMonth'"); //$birthDay and $birthMonth variablwes are already set at the top

foreach ($result as $row) {
    try {
        $mail->addAddress($row['email'], $row['fullname']);
    } catch (Exception $e) {
        echo 'Invalid address skipped: ' . htmlspecialchars($row['email']) . '<br>';
        continue;
    }

    try {
        $mail->send();
        echo 'Message sent to :' . htmlspecialchars($row['fullname']) . ' (' .
            htmlspecialchars($row['email']) . ')<br>';
        //Mark it as sent in the DB
        mysqli_query(
            $mysql,
            "UPDATE tablename SET sent = TRUE WHERE email = '" .
            mysqli_real_escape_string($mysql, $row['email']) . "'"
        );
    } catch (Exception $e) {
        echo 'Mailer Error (' . htmlspecialchars($row['email']) . ') ' . $mail->ErrorInfo . '<br>';
        //Reset the connection to abort sending this message
        //The loop will continue trying to send to the rest of the list
        $mail->getSMTPInstance()->reset();
    }
    //Clear all addresses and attachments for the next iteration
    $mail->clearAddresses();
    $mail->clearAttachments();
}
?>
```
