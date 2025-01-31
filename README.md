[![MIT license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT)
[![Packagist](https://img.shields.io/packagist/v/flownative/sentry.svg)](https://packagist.org/packages/flownative/sentry)
[![Maintenance level: Love](https://img.shields.io/badge/maintenance-%E2%99%A1%E2%99%A1%E2%99%A1-ff69b4.svg)](https://www.flownative.com/en/products/open-source.html)

# Sentry integration for Flow 8.x and 9.x

This [Flow](https://flow.neos.io) package allows you to automate
reporting of errors to [Sentry](https://www.sentry.io)

## Key Features

This package makes sure that throwables and exceptions logged in a Flow
application also end up in Sentry. This is done by implementing Flow's
`ThrowableStorageInterface` and configuring the default implementation.

This packages takes some extra care to  clean up paths and filenames of
stacktraces so you get a good overview while looking at an issue in the
Sentry UI.

## Installation

The Sentry integration is installed as a regular Flow package via
Composer. For your existing project, simply include `flownative/sentry`
into the dependencies of your Flow or Neos distribution:

```bash
$ composer require flownative/sentry
```

## Configuration

You need to at least specify a DSN to be used as a logging target. Apart
from that, you can configure the Sentry environment and release. These
options can either be set in the Flow settings or, more conveniently, by
setting the respective environment variables:

- `SENTRY_DSN`
- `SENTRY_ENVIRONMENT`
- `SENTRY_RELEASE`

The package uses these environment variables by default in the settings:

```yaml
Flownative:
  Sentry:
    dsn: "%env:SENTRY_DSN%"
    environment: "%env:SENTRY_ENVIRONMENT%"
    release: "%env:SENTRY_RELEASE%"
```

The error sample rate of Sentry can be set using

```yaml
Flownative:
  Sentry:
    sampleRate: 1
```

The default is 1 – 100% percent of all errors are sampled.

Throwables (that includes exceptions and runtime errors) are logged as
Sentry events. You may specify a list of exceptions which should not be
recorded. If such an exception is thrown, it will only be logged as a
"notice".

```yaml
Flownative:
  Sentry:
    capture:
      excludeExceptionTypes:
        - 'Neos\Flow\Mvc\Controller\Exception\InvalidControllerException'
```

If an ignored exception is handled by this Sentry client, it is logged
similar to the following message:

```
… NOTICE Exception 12345: The exception message (Ref: 202004161706040c28ae | Sentry: ignored)
```

## Additional Data

Exceptions declared in an application can optionally implement
`WithAdditionalDataInterface` provided by this package. If they do, the
array returned by `getAdditionalData()` will be visible in the "additional
data" section in Sentry.

Note that the array must only contain values of simple types, such as
strings, booleans or integers.

## Testing the Client

This package provides a command controller which allows you to log a
test message and a test exception.

Run the following command in your terminal to test your configuration:

```
./flow sentry:test
Testing Sentry setup …
Using the following configuration:
+-------------+----------------------------------------------------------+
| Option      | Value                                                    |
+-------------+----------------------------------------------------------+
| DSN         | https://abc123456789abcdef1234567890ab@sentry.io/1234567 |
| Environment | development                                              |
| Release     | dev                                                      |
| Server Name | test_container                                           |
| Sample Rate | 1                                                        |
+-------------+----------------------------------------------------------+

This command will now throw an exception for testing purposes.

Test exception in ThrowingClass

  Type: Flownative\Sentry\Test\SentryClientTestException
  Code: 1662712736
  File: Data/Temporary/Development/SubContextBeach/SubContextInstance/Cache/Code/Fl
        ow_Object_Classes/Flownative_Sentry_Test_ThrowingClass.php
  Line: 41

Nested exception:
Test "previous" exception in ThrowingClass

  Type: RuntimeException
  Code: 1662712735
  File: Data/Temporary/Development/SubContextBeach/SubContextInstance/Cache/Code/Fl
        ow_Object_Classes/Flownative_Sentry_Test_ThrowingClass.php
  Line: 40

Open Data/Logs/Exceptions/202411181211403b652e.txt for a full stack trace.
```

There are two more test modes for message capturing and error handling:

- `./flow sentry:test --mode message`
- `./flow sentry:test --mode error`
