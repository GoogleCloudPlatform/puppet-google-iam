# Google Cloud IAM Puppet Module

[![Puppet Forge](http://img.shields.io/puppetforge/v/google/giam.svg)](https://forge.puppetlabs.com/google/giam)

#### Table of Contents

1. [Module Description - What the module does and why it is useful](
    #module-description)
2. [Setup - The basics of getting started with Google Cloud IAM](#setup)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](
   #reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)

## Module Description

This Puppet module manages the resource of Google Cloud IAM.
You can manage its resources using standard Puppet DSL and the module will,
under the hood, ensure the state described will be reflected in the Google
Cloud Platform resources.

## Setup

To install this module on your Puppet Master (or Puppet Client/Agent), use the
Puppet module installer:

    puppet module install google-giam

Optionally you can install support to _all_ Google Cloud Platform products at
once by installing our "bundle" [`google-cloud`][bundle-forge] module:

    puppet module install google-cloud

## Usage

### Credentials

All Google Cloud Platform modules use an unified authentication mechanism,
provided by the [`google-gauth`][] module. Don't worry, it is automatically
installed when you install this module.

```puppet
gauth_credential { 'mycred':
  path     => $cred_path, # e.g. '/home/nelsonjr/my_account.json'
  provider => serviceaccount,
  scopes   => [
    'https://www.googleapis.com/auth/iam',
  ],
}
```

Please refer to the [`google-gauth`][] module for further requirements, i.e.
required gems.

### Examples

#### `giam_service_account`

```puppet
giam_service_account { 'test-account@graphite-playground.google.com.iam.gserviceaccount.com':
  ensure       => present,
  display_name => 'My Puppet test key',
  project      => 'google.com:graphite-playground',
  credential   => 'mycred',
}

```

#### `giam_service_account_key`

```puppet
giam_service_account_key { 'test-name':
  ensure           => present,
  service_account  => 'myaccount',
  path             => '/home/nelsona/test.json',
  key_algorithm    => 'KEY_ALG_RSA_2048',
  private_key_type => 'TYPE_GOOGLE_CREDENTIALS_FILE',
  project          => 'google.com:graphite-playground',
  credential       => 'mycred',
}

```


### Classes

#### Public classes

* [`giam_service_account`][]:
    A service account in the Identity and Access Management API.
* [`giam_service_account_key`][]:
    A service account in the Identity and Access Management API.

### About output only properties

Some fields are output-only. It means you cannot set them because they are
provided by the Google Cloud Platform. Yet they are still useful to ensure the
value the API is assigning (or has assigned in the past) is still the value you
expect.

For example in a DNS the name servers are assigned by the Google Cloud DNS
service. Checking these values once created is useful to make sure your upstream
and/or root DNS masters are in sync.  Or if you decide to use the object ID,
e.g. the VM unique ID, for billing purposes. If the VM gets deleted and
recreated it will have a different ID, despite the name being the same. If that
detail is important to you you can verify that the ID of the object did not
change by asserting it in the manifest.

### Parameters

#### `giam_service_account`

A service account in the Identity and Access Management API.


#### Example

```puppet
giam_service_account { 'test-account@graphite-playground.google.com.iam.gserviceaccount.com':
  ensure       => present,
  display_name => 'My Puppet test key',
  project      => 'google.com:graphite-playground',
  credential   => 'mycred',
}

```

#### Reference

```puppet
giam_service_account { 'id-of-resource':
  display_name     => string,
  email            => string,
  name             => string,
  oauth2_client_id => string,
  unique_id        => string,
  project_id       => string,
  project          => string,
  credential       => reference to gauth_credential,
}
```

##### `name`

  The name of the service account.

##### `display_name`

  User specified description of service account.


##### Output-only properties

* `project_id`: Output only.
  Id of the project that owns the service account.

* `unique_id`: Output only.
  Unique and stable id of the service account

* `email`: Output only.
  Email address of the service account.

* `oauth2_client_id`: Output only.
  OAuth2 client id for the service account.

#### `giam_service_account_key`

A service account in the Identity and Access Management API.


#### Example

```puppet
giam_service_account_key { 'test-name':
  ensure           => present,
  service_account  => 'myaccount',
  path             => '/home/nelsona/test.json',
  key_algorithm    => 'KEY_ALG_RSA_2048',
  private_key_type => 'TYPE_GOOGLE_CREDENTIALS_FILE',
  project          => 'google.com:graphite-playground',
  credential       => 'mycred',
}

```

#### Reference

```puppet
giam_service_account_key { 'id-of-resource':
  fail_if_mismatch  => boolean,
  key_algorithm     => 'KEY_ALG_UNSPECIFIED', 'KEY_ALG_RSA_1024' or 'KEY_ALG_RSA_2048',
  key_id            => string,
  name              => string,
  path              => string,
  private_key_data  => string,
  private_key_type  => 'TYPE_UNSPECIFIED', 'TYPE_PKCS12_FILE' or 'TYPE_GOOGLE_CREDENTIALS_FILE',
  public_key_data   => string,
  service_account   => reference to giam_service_account,
  valid_after_time  => time,
  valid_before_time => time,
  project           => string,
  credential        => reference to gauth_credential,
}
```

##### `private_key_type`

  Output format for the service account key.

##### `key_algorithm`

  Specifies the algorithm for the key.

##### `service_account`

  A reference to ServiceAccount resource

##### `path`

  The full name of the file that will hold the service account private
  key. The management of this file will depend on the value of
  sync_file parameter.
  File path must be absolute.

##### `key_id`

  Used to ensure the deletion of the key in the absence of a key file.

##### `fail_if_mismatch`

  If set to 'true' protects the target file from being rewritten with a
  new private key. By default the file is always ensured to have a valid
  private key on final state.


##### Output-only properties

* `name`: Output only.
  The name of the key.

* `private_key_data`: Output only.
  Private key data. Base-64 encoded.

* `public_key_data`: Output only.
  Public key data. Base-64 encoded.

* `valid_after_time`: Output only.
  Key can only be used after this time.

* `valid_before_time`: Output only.
  Key can only be used before this time.


## Limitations

This module has been tested on:

* RedHat 6, 7
* CentOS 6, 7
* Debian 7, 8
* Ubuntu 12.04, 14.04, 16.04, 16.10
* SLES 11-sp4, 12-sp2
* openSUSE 13
* Windows Server 2008 R2, 2012 R2, 2012 R2 Core, 2016 R2, 2016 R2 Core

Testing on other platforms has been minimal and cannot be guaranteed.

## Development

### Automatically Generated Files

Some files in this package are automatically generated by
[Magic Modules][magic-modules].

We use a code compiler to produce this module in order to avoid repetitive tasks
and improve code quality. This means all Google Cloud Platform Puppet modules
use the same underlying authentication, logic, test generation, style checks,
etc.

Learn more about the way to change autogenerated files by reading the
[CONTRIBUTING.md][] file.

### Contributing

Contributions to this library are always welcome and highly encouraged.

See [CONTRIBUTING.md][] for more information on how to get
started.

### Running tests

This project contains tests for [rspec][], [rspec-puppet][] and [rubocop][] to
verify functionality. For detailed information on using these tools, please see
their respective documentation.

#### Testing quickstart: Ruby > 2.0.0

```
gem install bundler
bundle install
bundle exec rspec
bundle exec rubocop
```

#### Debugging Tests

In case you need to debug tests in this module you can set the following
variables to increase verbose output:

Variable                | Side Effect
------------------------|---------------------------------------------------
`PUPPET_HTTP_VERBOSE=1` | Prints network access information by Puppet provier.
`PUPPET_HTTP_DEBUG=1`   | Prints the payload of network calls being made.
`GOOGLE_HTTP_VERBOSE=1` | Prints debug related to the network calls being made.
`GOOGLE_HTTP_DEBUG=1`   | Prints the payload of network calls being made.

During test runs (using [rspec][]) you can also set:

Variable                | Side Effect
------------------------|---------------------------------------------------
`RSPEC_DEBUG=1`         | Prints debug related to the tests being run.
`RSPEC_HTTP_VERBOSE=1`  | Prints network expectations and access.

[magic-modules]: https://github.com/GoogleCloudPlatform/magic-modules
[CONTRIBUTING.md]: CONTRIBUTING.md
[bundle-forge]: https://forge.puppet.com/google/cloud
[`google-gauth`]: https://github.com/GoogleCloudPlatform/puppet-google-auth
[rspec]: http://rspec.info/
[rspec-puppet]: http://rspec-puppet.com/
[rubocop]: https://rubocop.readthedocs.io/en/latest/
[`giam_service_account`]: #giam_service_account
[`giam_service_account_key`]: #giam_service_account_key
