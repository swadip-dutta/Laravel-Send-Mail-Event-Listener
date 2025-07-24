✅ First Step Create .env file and setup this (Demo)
MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

✅ Step 1: Mailable ক্লাস তৈরি করা

php artisan make:mail WelcomeMail

app/Mail/WelcomeMail.php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;

class WelcomeMail extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    public $user;

    public function __construct($user)
    {
        $this->user = $user;
    }

    public function build()
    {
        return $this->subject('Welcome to Our Website')
                    ->view('emails.welcome');
    }
}

✅ Step 2: Blade ইমেইল ভিউ তৈরি

resources/views/emails/welcome.blade.php

<!DOCTYPE html>
<html>
<head>
    <title>Welcome Email</title>
</head>
<body>
    <h1>Hello, {{ $user->name }}</h1>
    <p>Thanks for registering with us!</p>
</body>
</html>

✅ Step 3: Event তৈরি করা

php artisan make:event UserRegistered

app/Events/UserRegistered.php

namespace App\Events;

use App\Models\User;
use Illuminate\Queue\SerializesModels;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Broadcasting\InteractsWithSockets;

class UserRegistered
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }
}

✅ Step 4: Listener তৈরি করা

php artisan make:listener SendWelcomeEmail --event=UserRegistered

app/Listeners/SendWelcomeEmail.php

namespace App\Listeners;

use App\Events\UserRegistered;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use App\Mail\WelcomeMail;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use InteractsWithQueue;

    public function handle(UserRegistered $event): void
    {
        Mail::to($event->user->email)->send(new WelcomeMail($event->user));
    }
}

✅ Step 5: EventServiceProvider এ Register করা

app/Providers/EventServiceProvider.php

protected $listen = [
    \App\Events\UserRegistered::class => [
        \App\Listeners\SendWelcomeEmail::class,
    ],
];

✅ Step 6: Event Dispatch করা (যেমন Registration এর পরে)

যেকোনো জায়গা থেকে ইভেন্ট ট্রিগার করা যায়, যেমন:

use App\Events\UserRegistered;

$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password'),
]);

event(new UserRegistered($user));

✅Step 7: Queue Config (optional but recommended)

    .env ফাইলে QUEUE_CONNECTION=database বা redis সেট করুন।

    Queue worker চালাতে:

php artisan queue:work
