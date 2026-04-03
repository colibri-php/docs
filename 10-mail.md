# Mail

Colibri uses [PHPMailer](https://github.com/PHPMailer/PHPMailer) for email delivery, with Latte templates for email content.

## Configuration

```env
MAIL_DRIVER=log           # log, smtp, or sendmail
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USER=user@example.com
MAIL_PASS=secret
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=noreply@example.com
MAIL_FROM_NAME=MyApp
```

In development, use `MAIL_DRIVER=log` — emails are written to `storage/logs/mail.log` instead of being sent.

## Sending

### Simple

```php
use Colibri\Mail\Mail;

Mail::send(
    to: 'user@example.com',
    subject: 'Welcome!',
    template: base_path('templates/emails/welcome.latte'),
    data: ['name' => 'Alice'],
);
```

### Multiple Recipients

```php
Mail::send(
    to: ['user1@example.com', 'user2@example.com'],
    subject: 'Newsletter',
    template: base_path('templates/emails/newsletter.latte'),
    data: ['month' => 'April'],
);
```

### Advanced (CC, BCC, Attachments)

```php
$message = Mail::message(
    to: 'client@example.com',
    subject: 'Invoice #123',
    template: base_path('templates/emails/invoice.latte'),
    data: ['invoice' => $invoice],
);

$message->cc('manager@example.com')
    ->bcc('archive@example.com')
    ->replyTo('billing@example.com')
    ->attach('/path/to/invoice.pdf');

Mail::deliver($message);
```

## Email Templates

Email templates are Latte files in `templates/emails/`:

```latte
{* templates/emails/welcome.latte *}
{layout '../layouts/email.latte'}

{block content}
<h1>Welcome, {$name}!</h1>
<p>Thank you for joining us.</p>
{/block}
```

The email layout is at `templates/layouts/email.latte`:

```latte
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body style="font-family: -apple-system, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; color: #333;">
    {block content}{/block}
</body>
</html>
```

## Drivers

### `log` (default)

Writes emails to `storage/logs/mail.log`. No SMTP setup needed. Use for development.

### `smtp`

Sends via SMTP using PHPMailer. Configure host, port, username, password, and encryption in `.env`.

### `sendmail`

Sends via the server's sendmail binary.

### Custom Drivers

Implement `MailDriverInterface`:

```php
use Colibri\Mail\Interfaces\MailDriverInterface;
use Colibri\Mail\Message;

class SendGridMailDriver implements MailDriverInterface
{
    public function send(Message $message): bool
    {
        // SendGrid API logic...
        return true;
    }
}
```

Register it:

```php
Mail::registerDriver('sendgrid', SendGridMailDriver::class);
```

Set in `.env`:

```env
MAIL_DRIVER=sendgrid
```
