# Laravel FCM

# This package is deprecated new Package is https://github.com/prgayman/larafcm

## Introduction

Laravel FCM is an easy to use package working with both Laravel and Lumen for sending push notification with [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) (FCM).

It currently **only supports HTTP protocol** for :

- sending a downstream message to one or multiple devices
- create migrate to save user tokens
- add fucntion helpers to get or create user tokens

## Installation

To get the latest version of Laravel FCM on your project, require it from "composer":

    $ composer require prgayman/laravel-fcm

### Laravel

Register the provider directly in your app configuration file config/app.php `config/app.php`:

Laravel >= 5.5 provides package auto-discovery, thanks to rasmuscnielsen and luiztessadri who help to implement this feature in Laravel FCM, the registration of the provider and the facades should not be necessary anymore.

```php
'providers' => [
	Prgayman\Fcm\FcmServiceProvider::class,
]
```

Add the facade aliases in the same file:

```php
'aliases' => [

  'FCM' => Prgayman\Fcm\Facades\Fcm::class,

]
```

Publish the package config file using the following command:

    $ php artisan vendor:publish --provider="Prgayman\Fcm\FcmServiceProvider"

migrate fcm_tokens using the following command:

    $ php artisan migrate

### Lumen

Register the provider in your bootstrap app file `boostrap/app.php`

Add the following line in the "Register Service Providers" section at the bottom of the file.

```php
$app->register(Prgayman\Fcm\FcmServiceProvider::class);
```

For facades, add the following lines in the section "Create The Application" . FCMGroup facade is only necessary if you want to use groups message in your application.

```php
class_alias(\Prgayman\Fcm\Facades\Fcm::class, 'PFCM');
```

Copy the config file `fcm.php` manually from the directory `/vendor/prgayman/laravel-fcm/Config` to the directory `/config` (you may need to create this directory).

### Package Configuration

In your `.env` file, add the server key and the secret key for the Firebase Cloud Messaging:

```php
FIREBASE_SERVER_KEY=my_server_key
```

To get these keys, you must create a new application on the [firebase cloud messaging console](https://console.firebase.google.com/).

After the creation of your application on Firebase, you can find keys in `project settings -> cloud messaging`.

## Documentation

See the [Documentation](https://github.com/prgayman/laravel-fcm/wiki) for detailed installation and usage instructions.

#### Payload Builder
 - [NotificationBuilder](https://github.com/prgayman/laravel-fcm/wiki/NotificationBuilder)
 - [OptionsBuilder](https://github.com/prgayman/laravel-fcm/wiki/OptionsBuilder)
 - [DataBuilder](https://github.com/prgayman/laravel-fcm/wiki/DataBuilder)


## Basic Usage

Two types of messages can be sent using Laravel FCM:

- Notification messages, sometimes thought of as "display messages"
- Data messages, which are handled by the client app


First, add the Prgayman\Fcm\Traits\HasFcm trait to your User model(s):

```php

use Illuminate\Foundation\Auth\User as Authenticatable;
use Prgayman\Fcm\Traits\HasFcm; // Add this line

class User extends Authenticatable
{
    use HasFcm; // Add this line
}

$user = User::find(1);

/**
 * Get all fcm tokens this user
 * @param string $locale (optional)
 *
 * @return array
 */
$user->getFcmTokens();

/**
 * Get ios fcm tokens this user
 * @param string $locale (optional)
 *
 * @return array
 */
$user->getFcmIosTokens();

/**
 * Get android fcm tokens this user
 * @param string $locale (optional)
 *
 * @return array
 */
$user->getAndroidTokens();

/**
 * Get web fcm tokens this user
 * @param string $locale (optional)
 *
 * @return array
 */
$user->getWebTokens();

/**
 * Create Or Update user fcm Token this user
 * @param string $token
 * @param string $platform (optional)
 * @param string $locale (optional)
 *
 * @return Prgayman\Fcm\Models\FcmToken
 */
$fcmToken = $user->fcmCreateOrUpdate($token, "android", app()->getLocale()));

```

### Prgayman\Fcm\Models\FcmToken Model Usage

```php

use Prgayman\Fcm\Models\FcmToken;

$user = User::find(1);
/**
 * Get all fcm tokens
 * @param string $locale (optional)
 *
 * @return array
 */
$fcmToken = FcmToken::getTokens("en");

/**
 * Get Platform Ios Tokens
 * @param string $locale (optional)
 *
 * @return array
 */
$fcmToken = FcmToken::getIosTokens();

/**
 * Get Platform android Tokens
 * @param string $locale (optional)
 *
 * @return array
 */
$fcmToken = FcmToken::getAndroidTokens();

/**
 * Get Platform web Tokens
 * @param string $locale (optional)
 *
 * @return array
 */
$fcmToken = FcmToken::getWebTokens();

/**
 * Create Or Update Token
 * @param string $token
 * @param Illuminate\Database\Eloquent\Model $model_type (optional)
 * @param int $model_id (optional)
 * @param string $platform (optional)
 * @param string $locale (optional)
 *
 * @return Prgayman\Fcm\Models\FcmToken
 */
$fcmToken = FcmToken::createOrUpdate($token, get_class($user), $user->id );

```

### Downstream Messages

A downstream message is a notification message, a data message, or both, that you send to a target device or to multiple target devices using its registration_Ids.

The following use statements are required for the examples below:

```php
use Prgayman\Fcm\Support\Payload\Data\DataBuilder;
use Prgayman\Fcm\Support\Payload\Notification\NotificationBuilder;
use Prgayman\Fcm\Support\Payload\Options\OptionsBuilder;
use PFCM;
```

#### Sending a Downstream Message to a Device

```php

// Create Notification Builder
$notifyBuilder = new NotificationBuilder;
$notifyBuilder->setTitle("Notification Title");
$notifyBuilder->setBody("Notification Body");
$notifyBuilder->ios->setSubtitle("Sub title");
$notifyBuilder->setImage("image_url");
$notifyBuild = $notifyBuilder->build();

// Create Data Builder
$dataBuilder = new DataBuilder;
$dataBuilder->setData(["type" => "VIEW_PRODUCT"]);
$dataBuilder->addData(["type_id" => 1]);
$dataBuild = $dataBuilder->build();

// Create Options Builder
$optionsBuilder = new OptionsBuilder;
$optionsBuilder->setPriority("high"); // [normal|high]
$optionsBuilder->setContentAvailable(true);
$optionsBuild = $optionsBuilder->build();

// @var array|string
$tokens =[];

// Send Notification
$fcm = Fcm::sendNotification($tokens,$notifyBuild,$optionsBuild); // $optionsBuild (optional)

// Send Notification  With Data
$fcm = Fcm::sendNotificationWithData($tokens,$notifyBuild,$dataBuild,$optionsBuild); // $optionsBuild (optional)

// Send  Data
$fcm = Fcm::sendData($tokens,$dataBuild,$optionsBuild); // $optionsBuild (optional)

// return in number success send notification
$numberSuccess = $fcm->numberSuccess();

// return in number failure send notification
$numberFailure = $fcm->numberFailure();

// return in number modification send notification
$numberModification = $fcm->numberModification();

// return Array - you must remove all this tokens in your database
$tokensToDelete = $fcm->tokensToDelete();

// return Array (key : oldToken, value : new token - you must change the token in your database)
$tokensToModify = $fcm->tokensToModify();

// return Array - you should try to resend the message to the tokens in the array
$tokensToRetry= $fcm->tokensToRetry();

// return Array (key:token, value:error) - in production you should remove from your database the tokens
$tokensWithError = $fcm->tokensWithError();

```

## Licence

This library is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).

Some of this documentation is coming from the official documentation. You can find it completely on the [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) Website.
