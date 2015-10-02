# Installation of the RealMe module

The module is best installed via Composer. Ensure the CWP Composer repository is added to your composer.json. We also 
need to specify a special version of SimpleSAMLphp and the SAML2 parsing library which contain fixes specific to using 
SimpleSAMLphp with SilverStripe. Eventually, it's hoped that these will not be necessary, but that requires code being 
merged into SimpleSAMLphp.

```json
"repositories": [
    {
        "type": "composer",
        "url": "https://packages.cwp.govt.nz/"
    },
    {
    	"type": "vcs",
    	"url": "https://github.com/madmatt/simplesamlphp.git"
    },
    {
    	"type": "vcs",
    	"url": "https://github.com/madmatt/saml2.git"
    }
```

Then include the package:

```bash
composer require silverstripe/silverstripe-realme
```

After composer installation, you need to modify your `.htaccess` file in your web-root, in order to allow access to the 
SimpleSAMLphp web-root - normally this is not allowed as it's within the 'vendor' directory, however SimpleSAMLphp 
requires this.

Normally, you'd have something like the following:
```
<IfModule mod_rewrite.c>
	SetEnv HTTP_MOD_REWRITE On
	RewriteEngine On

	# Enable HTTP Basic authentication workaround for PHP running in CGI mode
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

	# Deny access to potentially sensitive files and folders
	RewriteRule ^vendor(/|$) - [F,L,NC]
	RewriteRule silverstripe-cache(/|$) - [F,L,NC]
	RewriteRule composer\.(json|lock) - [F,L,NC]

	# Redirect legacy index.php requests
	RewriteCond %{REQUEST_URI} ^(?:(.*)/)?index\.php [NC]
	RewriteRule ^index\.php(?:/(.*))?$ %1/$1 [R=301,L]

	# Process through SilverStripe if no file with the requested name exists.
	# Pass through the original path as a query parameter, and retain the existing parameters.
	RewriteCond %{REQUEST_URI} ^(.*)$
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule .* framework/main.php?url=%1 [QSA]
</IfModule>
```

Just prior to the 'vendor' lines above, insert the following:
```
	# Allow access to SimpleSAMLphp directory within vendor/
	RewriteRule ^vendor/simplesamlphp/simplesamlphp/www - [L,NC]
```

Once installation is completed, configuration is required before this module will work. See the 
[configuration section](configuration.md) for full details.