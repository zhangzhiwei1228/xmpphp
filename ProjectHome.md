XMPPHP is the successor to Class.Jabber.PHP that I've been promising for years.  Taking advantage of PHP5, I believe it to be an elegant solution with a direct approach.

Some of the features include:
  * Connect to any XMPP 1.0 server (Google Talk, LJ Talk, jabber.org, etc)
  * Supports TLS encryption
  * Several XML processing approaches and supported styles (process indefinitely, processUntil an event, processTime for a number of seconds), waiting on events or map them, etc.

**I don't host the SVN @ Google, so get it at svn://netflint.net/xmpphp .**  I encourage you to check out my python XMPP Library (SleekXMPP) at http://code.google.com/p/sleekxmpp .

Feel fee to IM me any time with questions, feature requests, etc.  Don't be afraid to take a little bit of my time!

**Jabber ID**: fritzy AT netflint.net

**Jabber MUC (Chatroom)**:  xmpphp@conference.psi-im.org

# Known Issues #
  * Due to PHP bug http://bugs.php.net/bug.php?id=42682, possibly due to a GCC optimization bug, this library might not work on PHP compiled for x86\_64 (Fixed in CVS and PHP 5.2.6).
  * TLS encrypted connections lock when more than a few k of data comes in without line breaks. This happens on some servers when collecting vCards and the like. This appears to be a PHP bug that I will file soon.

# Examples #
A simple code example for sending messages:
```
<?php
include("xmpp.php");
$conn = new XMPP('talk.google.com', 5222, 'username', 'password', 'xmpphp', 'gmail.com', $printlog=False, $loglevel=LOGGING_INFO);
$conn->connect();
$conn->processUntil('session_start');
$conn->message('someguy@someserver.net', 'This is a test message!');
$conn->disconnect();
?>
```
To not use SSL/TLS encryption if available, set
```
$conn->use_encryption = False;
```
before calling connect()

A command line bot example:
```
<?php
include("xmpp.php");
$conn = new XMPP('talk.google.com', 5222, 'user', 'password', 'xmpphp', 'gmail.com', $printlog=True, $loglevel=LOGGING_INFO);
$conn->connect();
while(!$conn->disconnected) {
    $payloads = $conn->processUntil(array('message', 'presence', 'end_stream', 'session_start'));
    foreach($payloads as $event) {
        $pl = $event[1];
        switch($event[0]) {
            case 'message':
                print "---------------------------------------------------------------------------------\n";
                print "Message from: {$pl['from']}\n";
                if($pl['subject']) print "Subject: {$pl['subject']}\n";
                print $pl['body'] . "\n";
                print "---------------------------------------------------------------------------------\n";
                $conn->message($pl['from'], $body="Thanks for sending me \"{$pl['body']}\".", $type=$pl['type']);
                if($pl['body'] == 'quit') $conn->disconnect();
                if($pl['body'] == 'break') $conn->send("</end>");
            break;
            case 'presence':
                print "Presence: {$pl['from']} [{$pl['show']}] {$pl['status']}\n";
            break;
            case 'session_start':
                $conn->presence($status="Cheese!");
            break;
        }
    }
}
?>
```

# To Do #
  * SRV DNS Lookups