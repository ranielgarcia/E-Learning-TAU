# E-Learning TAU

E-Learning system for Tarlac Agriculture University (Agriculture Department), built with **CodeIgniter 3.1.9**.

> ⚠️ This project was built ~8 years ago on **CodeIgniter 3.1.9**, a legacy PHP framework version.
> It is **not compatible with PHP 8+**. You must run it on an older PHP version (see requirements below).

## Tech Stack

- PHP (CodeIgniter 3.1.9 — bundled under `system/`, not installed via Composer)
- MySQL / MariaDB
- Apache with `mod_rewrite` (e.g. via XAMPP)

## Requirements

- **XAMPP with PHP 5.6, 7.0, 7.1, 7.2, 7.3 or 7.4** — CodeIgniter 3.1.9 will fatal-error on PHP 8+ (deprecated/removed functions such as `each()`, old-style constructors, etc.). Modern XAMPP installs default to PHP 8+, so you need an older XAMPP bundle or a separate PHP 7.x install.
- MySQL/MariaDB (bundled with XAMPP)
- Apache `mod_rewrite` enabled

## Setup Instructions

1. **Get the code**
   Copy/clone this repository into your web server root, e.g. `C:\xampp\htdocs\elearning`.

2. **Use a compatible PHP version**
   Install/switch XAMPP (or your PHP runtime) to PHP 5.6–7.4. Verify with:
   ```
   php -v
   ```

3. **Create the database**
   Import the schema in [database-query/query.sql](database-query/query.sql), which creates the `ELearning_db` database and its tables:
   ```
   mysql -u root -p < database-query/query.sql
   ```
   or import the same file through phpMyAdmin.

4. **Configure the database connection**
   Edit [application/config/database.php](application/config/database.php) if your local credentials differ from the defaults:
   ```php
   'hostname' => 'localhost',
   'username' => 'root',
   'password' => '',
   'database' => 'ELearning_db',
   'dbdriver' => 'mysqli',
   ```

5. **Configure the base URL**
   Edit [application/config/config.php](application/config/config.php) and set `$config['base_url']` to match where you placed the project, e.g.:
   ```php
   $config['base_url'] = 'http://localhost/elearning/';
   ```

6. **Enable mod_rewrite**
   The root [.htaccess](.htaccess) rewrites clean URLs to `index.php`. Make sure Apache has `mod_rewrite` enabled and `AllowOverride All` set for the project directory.

7. **(Recommended) Set an encryption key**
   `$config['encryption_key']` in `application/config/config.php` is empty by default. Set it to a random string for session/security best practices.

8. **Start the server**
   Start Apache and MySQL (e.g. via the XAMPP control panel), then browse to the base URL configured in step 5.

## Project Structure

- `application/` — MVC app code (controllers, models, views, config)
- `system/` — CodeIgniter 3.1.9 core framework (do not modify)
- `assets/` — CSS, JS, images, and third-party front-end plugins
- `database-query/` — database schema (`query.sql`) and diagrams

## Troubleshooting

- **Fatal errors / blank page on PHP 8+**: switch to PHP 5.6–7.4, CodeIgniter 3.1.9 is not PHP 8 compatible.
- **404 on every page except the homepage**: confirm `mod_rewrite` is enabled and `.htaccess` is being read (`AllowOverride All`).
- **Database connection errors**: double-check `application/config/database.php` matches your local MySQL credentials and that `ELearning_db` was imported successfully.
