# Drupal Security Audit

## About
drupal-security-audit is a set of [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) rules that finds vulnerabilities and weaknesses related to security in PHP code.

It currently has core PHP rules as well as Drupal 7 specific rules.

The tool also checks for CVE issues and security advisories related to CMS/framework. Using it, you can follow the versioning of components during static code analysis.

The main reason of this project for being an extension of PHP_CodeSniffer is to have easy integration into continuous integration systems. It is also able to find security bugs that are not detected with object oriented analysis (like in [RIPS](http://rips-scanner.sourceforge.net/) or [PHPMD](http://phpmd.org/)).

## Installation

First, make sure Composer is installed correctly:

    which composer

If you get composer not found or similar, follow Composer's installation
instructions.

Install Coder (8.x-2.x) in your global Composer directory in your home directory
(`~/.composer`):

    composer global require thereference/drupal-security-audit

To make the `phpcs` and `phpcbf` commands available globally, add the Composer
bin path to your `$PATH` variable in `~/.profile`, `~/.bashrc` or `~/.zshrc`:

    export PATH="$PATH:$HOME/.composer/vendor/bin"

Register the Drupal and DrupalPractice Standard with PHPCS:

    phpcs --config-set installed_paths ~/.composer/vendor/thereference/drupal-security-audit//coder_sniffer

### Composer Installer Plugins

The Coder package (>= 8.2.11) now works with Composer Installer Plugins,
that find and register standards whenever packages are installed or updated.
To use such a plugin within your project, follow these steps.

    composer require --dev dealerdirect/phpcodesniffer-composer-installer
    composer require --dev thereference/drupal-security-audit

Now, you will see Drupal and DrupalPractice listed in the available PHP
CodeSniffer standards.

    vendor/bin/phpcs -i

The same can be done for a Composer global installation.

    composer global require dealerdirect/phpcodesniffer-composer-installer
    composer global require thereference/drupal-security-audit


Usage
-----

Simply point to any XML ruleset file and a folder:
```
phpcs --extensions=php,inc,lib,module,info --standard=Drupal7Security /your/php/files/
```

Specifying extensions is important since for example PHP code is within .module files in Drupal.

To have a quick example of output you can use the provided tests.php file:
```
$ phpcs --extensions=php,inc,lib,module,info --standard=Drupal7Security.xml tests.php

FILE: tests.php
--------------------------------------------------------------------------------
FOUND 16 ERROR(S) AND 15 WARNING(S) AFFECTING 22 LINE(S)
--------------------------------------------------------------------------------
  6 | WARNING | Possible XSS detected with . on echo
  6 | ERROR   | Easy XSS detected because of direct user input with $_POST on
    |         | echo
  8 | WARNING | db_query() is deprecated except when doing a static query
  8 | ERROR   | Potential SQL injection found in db_query()
  9 | WARNING | Usage of preg_replace with /e modifier is not recommended.

```

#### Drupal note

For the Drupal AdvisoriesContrib you need to change your `/etc/php5/cli/php.ini` to have:
```
short_open_tag = On
```
in order to get rid of "No PHP code was found in this file" warnings.

Please note that only Drupal modules downloaded from drupal.org are supported. If you are using contrib module but from another source, the version checking will probably won't work and will generate warning.


Customize
---------
As in normal PHP CodeSniffer rules, customization is provided in the XML files that are in the top folder of the project.

These global parameters are used in many rules:
* ParanoiaMode: set to 1 to add more checks. 0 for less.
* CmsFramework: set to the name of a folder containings rules and Utils.php (such as Drupal7, Symfony2).

They can be setted in the XML files or in command line for permanent config with `--config-set` or at runtime with `--runtime-set`. Note that the XML override all CLI options so remove it if you want to use it. The CLI usage is as follow `phpcs --runtime-set ParanoiaMode 0 --extensions=php --standard=example_base_ruleset.xml tests.php`;

In some case you can force the paranoia mode on or off with the parameter `forceParanoia` inside the XML rule.


Specialize
----------

If you want to fork and help or just do your own sniffs you can use the utilities provided by phpcs-security-audit rules in order to facilitate the process.

Let's say you have a custom CMS function that is taking user input from `$_GET` when a function call to `get_param()` is done.

You have to create a new Folder in Sniffs/ that will be the name of your framework. Then you'll need
to create a file named Utils.php that will actually be the function that will specialise the generic sniffs. To guide you, just copy the file from another folder such as Drupal7/.

The main function you'll want to change is `is_direct_user_input` where you'll want to return TRUE when `get_param()` is seen:
```php
	public static function is_direct_user_input($var) {
		if (parent::is_direct_user_input($var)) {
			return TRUE;
		} else {
			if ($var == 'get_param') {
				return TRUE;
			}
		}
		return FALSE;
	}
```

Don't forget to set the occurrence of param "CmsFramework" in your XML base configuration in order to select your newly added utilities.

You are not required to do your own sniffs for the modification to be useful, since you are specifying what is a user input for other rules, but you could use the newly created directory to do so.

If you implement any public cms/framework customization please make a pull request to help the project grows.


Annoyances
----------

As any security tools, this one comes with it's share of annoyance. At first a focus on finding vulnerabilities will be done, but later it is planned to have a phase where efforts will be towards reducing annoyances, in particular with the number of false positives.

* It's a generator of false positives. This can actually help you learn what are the weak functions in PHP. Paranoia mode will fix that by doing a major cut-off on warnings when set to 0.
* It's slow. On big Drupal modules and core it can take too much time (and RAM, reconfigure cli/php.ini to use 512M if needed) to run. Not sure if it's because of bugs in PHPCS or this set of rules, but will be investigated last. Meanwhile you can configure PHPCS to ignore big contrib modules (and run another instance of PHPCS for .info parsing only for them). An example is og taking hours, usually everything runs under 1-2 minutes and sometime around 5 minute. You can only use one core in PHP since no multithreading is available. Possible workaround is to use phpcs --ignore=folder to skip scanning of those parts.
* For Drupal advisories checking: a module with multiple versions might be secure if a lesser fixed version exists and you'll still get the error or warning. Keep everything updated at latest as recommended on Drupal's website.



