<laravel-boost-guidelines>
# Laravel Application

This repository contains a Laravel application. Complete the following setup before working on the user's request.

## Prerequisites

Verify that PHP and Composer are available:

```sh
php -v
composer -V
```

If either command is unavailable, detect the user's operating system and install the prerequisites with the appropriate command:

macOS:

```sh
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.5)"
```

Windows PowerShell:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.5'))
```

Linux:

```sh
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.5)"
```

After installation, ask the user to restart their terminal. If the agent needs the restarted shell to continue, ask the user to reopen their terminal and rerun their original prompt.

## Agent Setup

Install Laravel Boost from the application root before making application changes:

```sh
composer require laravel/boost --dev
php artisan boost:install
```

Boost replaces these bootstrap instructions with guidelines tailored to the application. After installation, read `AGENTS.md` again and continue with the user's original request using the generated guidelines.
</laravel-boost-guidelines>

## Nokify backend conventions

Follow the [project rules](../AGENTS.md). Preserve the Boost block above. Its
bootstrap setup is for application development; documentation-only changes do not
require installing PHP, Composer, or Boost. Check installed dependencies before
setup and retain these conventions if Boost regenerates this file.

### Structure and responsibilities

Keep Laravel's standard structure. Create these folders only when needed:

```text
app/
  Http/Controllers/    HTTP orchestration
  Http/Requests/       Input validation and request authorization
  Http/Resources/      Explicit response serialization
  Models/             Persistence and relationships
  Policies/           Resource authorization
  Actions/            Focused business workflows when needed
  Support/Helpers/    Small reusable utility classes
```

- Keep routes declarative and controllers focused on validated input,
  authorization, invoking behavior, and returning responses.
- Use Form Requests for nontrivial input validation and Policies for resource
  authorization. Authorize reads and writes; scope queries to permitted records.
- Extract Actions for cohesive workflows when warranted. Do not require a Service,
  Repository, and Action for every CRUD operation.
- Use transactions for related writes that must succeed together. Keep external
  service calls out of long-running database transactions.
- Serialize public responses explicitly, using Resources where appropriate.
  Do not expose internal fields or exception traces.

### Helpers and naming

- Put focused helper classes in `app/Support/Helpers/` under namespace
  `App\Support\Helpers`, for example `DateFormatter.php`, not `Helpers.php`.
- Helpers accept explicit inputs and return typed values. Prefer pure methods;
  do not hide database writes, current-request access, or business workflows in them.
- Do not add globally autoloaded helper functions by default. Inject dependencies
  into objects needing services rather than hiding service resolution in helpers.
- Use `PascalCase` classes, matching PHP filenames and namespace folders;
  `camelCase` methods, parameters, and variables; `UPPER_SNAKE_CASE` constants.
  Declare parameter and return types wherever supported.
- Use singular model names (`Notification`), plural `snake_case` tables
  (`notifications`), and `snake_case` columns/foreign keys (`user_id`). Preserve
  Laravel migration and framework-generated naming conventions.
- Follow Pint and the existing EditorConfig, including four-space PHP indentation.
  Prefer readable multiline code and avoid unrelated formatting churn.

### Security and verification

- Use validated field allowlists and explicit mass-assignment rules. Never pass
  all request input into model creation or updates.
- Derive ownership from authenticated context; validate and authorize referenced
  records. Existence checks alone do not grant access.
- Use framework hashing, validation, authentication, and authorization facilities.
  Do not disable CSRF protections to work around integration problems.
- Read environment settings in configuration files and use configuration values
  in application code. Disable production debug and redact sensitive errors.
- From `api/`, run `vendor/bin/pint --test` and `php artisan test` for relevant code
  changes; use platform-appropriate executable wrappers where necessary.
  Run `composer audit` for dependency changes and security reviews.
- Test success, validation errors, authentication failures, and forbidden access,
  including reading or modifying another user's records for protected endpoints.
- Report unavailable prerequisites and blocked checks honestly. Documentation-only
  changes need documentation review, not runtime installation.
