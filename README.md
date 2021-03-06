# Net SMTP

SMTP connector using Net_SMTP PEAR library.

## But why?

Drupal contrib already has a nice SMTP module called "SMTP" which can be found
at the following URL:

    http://www.drupal.org/project/smtp

But in real life, this modules attempts to use the PHPMailer library to connect
to the SMTP server, which can found at:

    https://github.com/PHPMailer/PHPMailer

If you look at it a bit more, you'll see that PHPMailer is not an SMTP
connector, while it can, its main goal is to format the MIME messages for
you.

Whenever you use Drupal with a module such as MIMEMail which can be found at:

    http://www.drupal.org/project/mimemail

you'll notice that your messages are already well formatted in a very precise
and valid MIME enveloppe.

*What happens behind this scenario is that the SMTP module needs to deconstruct
the valid MIME encoded message in order to be able to use the PHPMailer API
which then will attempt to rebuild a MIME message.*

In real life, it does not, it does deconstrut your MIME encoded message, but in
a very wrong way, and breaks it in a lot cases.

## Runtime configuration

### Drupal mail system configuration

For your convenience, this module uses a specific Drupal mail system
implementation that acts as a proxy which lets you use different
formatter and mailer.

In order to use it, simply add to your settings.php file:

    $conf['mail_system'] = array(
      'default-system' => 'NetSmtp_MailSystemProxy'
    );

Then you can set the formatter this way:

    $conf['netsmtp_proxy'] = array(
      'default' => 'MimeMailSystem',
    );

If you need to specify specific formatters for other modules, you
can use this variable the exact same way you would use the core
'mail_system' variable:

    $conf['netsmtp_proxy'] = array(
      'default' => 'MimeMailSystem',
      'MYMODULE' => 'SomeOtherFormatter',
      'MYMODULE_MYMAIL' => 'YetAnotherFormatter',
    );

And that's pretty much it.

### SMTP configuration

At minima you would need to specify your SMTP server host:

    $conf['netsmtp'] = array(
      'default' => array(
        'hostname' => '1.2.3.4'
      ),
    );

Hostname can be an IP or a valid hostname.

In order to work with SSL, just add the 'use_ssl' key with true or false.

You can set the port if you wish using the 'port' key.

If you need authentication, use this:

    $conf['netsmtp'] = array(
      'default' => array(
        'hostname' => 'smtp.provider.net',
        'username' => 'john',
        'password' => 'foobar',
      ),
    );

And additionnaly, if you need to advertise yourself as a different hostname
than the current localhost.localdomain, you can set the 'localhost' variable.

An complete example:

    $conf['netsmtp'] = array(
      'default' => array(
        'hostname'  => 'smtp.provider.net',
        'port'      => 465,
        'use_ssl'   => true,
        'username'  => 'john',
        'password'  => 'foobar',
        'localhost' => 'host.example.tld',
      ),
    );

Note that for now this only supports the PLAIN and LOGIN authentication
methods, I am definitly too lazy to include the Auth_SASL PEAR package
as well.

Additionnaly, you can change the 'use_ssl' paramater to the 'tls' value
instead, and hope for the best to happen, it should force the Net::SMTP
library to do a TLS connection instead.

### Advanced SMTP configuration

Additionnaly you can define a set of servers, for example if you need a
mailjet or mandrill connection:

    $conf['netsmtp'] = array(
      'default' => array(
        'host' => '1.2.3.4',
        'ssl'  => true,
      ),
      'mailjet' => array(
        'host' => '1.2.3.4',
        'ssl'  => true,
        'user' => 'john',
        'pass' => 'foobar',
      ),
    );

You can then force mails to go throught another server than default by
setting the 'smtp_provider' key in the Drupal $message array when sending
mail.

### Overriding the proxy

Additionnaly, if you have specific business needs, you can override the
proxy class, start by writing your own such as:

    MyProxy extends NetSmtp_MailSystemProxy
    {
        // Do something
    }

Then register it in any autoloader.

Then, tell the net-smtp module to use it instead of the stock provided one
into your settings.php file:

    $conf['mail_system'] = array(
      'default-system' => 'MyProxy'
    );

### Additional configuration

Per default this module uses the Drupal native function correctly encode
mail subjects, if you use a formatter that does the job for you, set
the _netsmtp\_subject\_encode_ to false to deactivate this behavior:

    $conf['netsmtp_subject_encode'] = false;

### Debugging

#### Re-route all outgoing mail

This feature is useful when working in a development phase where you don't
want mails to be sent to their real recipients. In order to activate it
just set:

    $conf['netsmtp_catch'] = 'someuser@example.com';

Moreover, you can set multiple recipients:

    $conf['netsmtp_catch'] = array(
      'user1@example.com',
      'user2@example.com',
      'user3@example.com',
      // ...
    );

Be careful that this is a debug feature and the recipient user addresses
won't be processed in any way, which means that you can set a mail address
containing a ',' character, it won't be escaped.

#### Sent data dumping

Additionally you can enable a debug output that will dump all MIME encoded
messages this module will send onto the file system. Just set:

    $conf['netsmtp_debug_mime'] = true;

And every mail will be dumped into the following Drupal temp folder:

    temporary://netsmtp/YYYY-MM-DD/

Additionnaly you can change the path using this variable:

    $conf['netsmtp_debug_mime_path'] = 'private://netsmtp';

#### Sent mail trace

This probably should belong to another module, but if you need extensive mail
tracing logging, you can enable:

    $conf['netsmtp_debug_trace'] = true;

This will activate a _hook\_mail\_alter()_ implementation that will log every
mail activity sent by the plateform in a single file:

    temporary://netsmtp/netsmtp-trace-YYYY-MM-DD.log

In this file you'll find various internal Drupal modules information about the
mails being sent, including the stack trace at the time the mail is beint sent.

Additionnaly you can change the path using this variable:

    $conf['netsmtp_debug_trace_path'] = 'private://netsmtp';

## Bundled libraries

PEAR and other libraries are included into this module for the sake of
simplicity.

### PEAR

PEAR v1.9.5 - Licensed under The New BSD License

Note that if you have a PHP compiled with the PEAR option native one will
be used instead. This should not be a problem.

### Net_Socket

Net_Socket v1.0.14 - Licensed under the PHP Licence v3.01

### Net_SMTP 

Net_SMTP v1.6.2 - Licensed under the PHP Licence v3.01

## Note about the autoloader

Feel free to provide your own version of bundled libraries, especially
if you need some other modules to share it. You only need to replace this
module autoloader with your own.

If you experience any problems for replacing this module autoloader,
just set the 'netsmtp_autoload_disable' variable to true.
