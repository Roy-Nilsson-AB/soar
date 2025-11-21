# Soar - Docker Compose Helper for PHP Development

## What is Soar?

Soar is a bash-based wrapper around Docker Compose designed to simplify development workflows for PHP applications, particularly Symfony and Magento projects. It provides convenient shortcuts for common development tasks while maintaining full access to Docker Compose functionality.

**Author**: Roy Nilsson (roy.nilsson@rnab.net)
**License**: MIT
**Repository**: https://gitlab.com/rnab-other/soar

## Core Philosophy

- **Wrapper, not replacement**: Unknown commands are passed directly to docker-compose
- **Convention over configuration**: Uses sensible defaults with environment variable overrides
- **Multi-framework support**: Works with Symfony, Magento, and generic PHP applications
- **Environment-aware**: Automatically sources `.env` and `.env.$APP_ENV` files

## Installation

Soar is installed globally via Composer:

```bash
composer global require rnab/soar
```

The `soar` binary is installed to `~/.composer/vendor/bin/soar` and should be in your PATH.

## Architecture & Configuration

### Environment Variables

Soar sources environment files in this order:
1. `./.env` (if exists)
2. `./.env.$APP_ENV` (if `APP_ENV` is set and file exists)

### Service Name Defaults

These can be overridden via environment variables:

- `DOCKER_PHP_SERVICE` - Default: `php`
- `DOCKER_DB_SERVICE` - Default: `db`
- `DOCKER_WEB_SERVER_SERVICE` - Default: `nginx`

### User Configuration

- `APP_USER` - If set, commands run as this user inside containers (via `--user=$APP_USER`)
- `USER` and `GROUP` - Automatically set from current system user if not defined

## Command Reference

### Docker Compose Commands

Any unrecognized command is passed directly to docker-compose:

```bash
soar up -d              # Start services in background
soar stop               # Stop services
soar restart            # Restart services
soar ps                 # List containers
soar logs               # View logs
soar down               # Stop and remove containers
```

### Special Commands

#### stopall
Stops ALL Docker containers on the system (not just project containers):

```bash
soar stopall
# Equivalent to: docker stop $(docker ps -a -q)
```

**⚠️ Warning**: This stops containers from ALL projects, not just the current one.

#### shell [container]
Opens an interactive shell in a container (defaults to PHP service):

```bash
soar shell              # Opens shell in php container
soar shell nginx        # Opens shell in nginx container
```

Automatically detects and uses `bash` if available, falls back to `sh`.

#### Symfony Console

```bash
soar console <command> [args]
# Example: soar console cache:clear
# Runs: docker-compose exec -it php ./bin/console <command>
```

Runs with `-it` flags for interactive TTY allocation (preserves colors).

#### Magento Commands

**Basic magento command:**
```bash
soar magento <command> [args]
# Example: soar magento cache:flush
# Runs: docker-compose exec -it php ./bin/magento <command>
```

**Special magento install:**
```bash
soar magento install
```

Runs Magento setup with values from environment variables:
- `MYSQL_HOST`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`
- `MAGENTO_ADMIN_USERNAME`, `MAGENTO_ADMIN_PASSWORD`, `MAGENTO_ADMIN_EMAIL`
- `MAGENTO_ADMIN_FIRSTNAME`, `MAGENTO_ADMIN_LASTNAME`
- `OPENSEARCH_HOST`

**Magento clean:**
```bash
soar magento clean
```

Removes all Magento generated files:
- `var/cache/`, `var/page_cache/`, `var/view_preprocessed`
- `var/di`, `var/generation`, `generated`
- `pub/static/frontend/`, `pub/static/adminhtml/`

#### Composer

```bash
soar composer <command> [args]
# Example: soar composer install
# Runs: docker-compose exec -it php composer <command>
```

#### Vendor Bin

```bash
soar bin <binary> [args]
# Example: soar bin phpunit
# Runs: docker-compose exec -it php ./vendor/bin/<binary>
```

#### Database Operations

**Interactive MySQL client:**
```bash
soar db
# Connects with: -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE
```

**Import database:**
```bash
soar db import <file.sql|file.sql.gz>
# Automatically detects .gz files and uses zcat
# Uses pv for progress bar if available
# Imports as root user: -uroot -p$MYSQL_ROOT_PASSWORD
```

**Export/dump database:**
```bash
soar db dump [args]
# Example: soar db dump > backup.sql
# Runs: mysqldump -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE
```

**Execute SQL query:**
```bash
soar db exec "<query>" [args]
# Example: soar db exec "SELECT * FROM users"
# Runs: mysql -e "<query>"
```

**Export to CSV:**
```bash
soar db csv "<query>" [args]
# Converts tab-delimited output to semicolon-delimited
```

## Key Implementation Details

### Interactive TTY Allocation

Recent feature addition: Symfony console and similar commands now use `-it` flags to preserve colors and interactive features:

```bash
$DOCKER_COMPOSE_EXEC -it $DOCKER_PHP_SERVICE ./bin/console $@
```

This ensures commands like `bin/console` maintain proper formatting and color output.

### Database Import as Root

Database imports use `root` user to avoid permission issues:

```bash
mysql -uroot -p$MYSQL_ROOT_PASSWORD $MYSQL_DATABASE
```

Regular database operations use `MYSQL_USER` credentials.

### Platform Detection

Soar verifies the operating system at startup:
- ✅ Linux
- ✅ macOS (Darwin)
- ❌ Other systems (exits with error)

### Color Support

Automatically detects terminal color support and uses colors if available:
- **YELLOW** - Section headers
- **GREEN** - Command examples
- **BOLD** - Emphasis

### Docker Compose Binary Detection

Automatically detects whether to use `docker compose` (newer) or `docker-compose` (legacy):

```bash
if docker compose &> /dev/null; then
    DOCKER_COMPOSE="docker compose"
else
    DOCKER_COMPOSE="docker-compose"
fi
```

## Environment Variable Requirements

### Database Operations
- `MYSQL_USER` - Database user
- `MYSQL_PASSWORD` - Database password
- `MYSQL_DATABASE` - Database name
- `MYSQL_ROOT_PASSWORD` - Root password (for imports)

### Magento Setup
- `MYSQL_HOST` - Database host
- All database vars above
- `MAGENTO_ADMIN_*` - Admin account details
- `OPENSEARCH_HOST` - Search engine host

## Important Notes

### Command Pass-through

Any command not explicitly handled by soar is passed to docker-compose:

```bash
soar build              # docker-compose build
soar pull               # docker-compose pull
soar exec php php -v    # docker-compose exec php php -v
```

### Framework Detection

The script automatically detects if Magento is available by checking:

```bash
docker-compose exec -T php bin/magento -V >/dev/null
```

This determines whether to show Magento commands in help output.

### Verification Steps Before Using Commands

Some commands mentioned in the documentation guide may not actually exist in the source:

**NOT implemented in source code:**
- `soar env` - No such command
- `soar stats` - No such command
- `soar config` - Would pass through to docker-compose config
- `soar php` - Would need to use `soar shell` then run php
- `soar exec` - Would pass through to docker-compose exec

**Correct alternatives:**
- Environment: Check `.env` files directly
- Stats: Use `docker stats` directly
- Config: Use `soar config` (passes to docker-compose)
- PHP commands: Use `soar shell` or docker-compose exec

## Common Patterns

### Daily Development

```bash
# Start environment
soar up -d

# Clear Symfony cache
soar console cache:clear

# Run migrations
soar console doctrine:migrations:migrate

# Database access
soar db
```

### Magento Development

```bash
# Start environment
soar up -d

# Install Magento
soar magento install

# Clear cache
soar magento cache:flush

# Reindex
soar magento indexer:reindex

# Deep clean
soar magento clean
```

### Database Workflows

```bash
# Backup
soar db dump > backup-$(date +%Y%m%d).sql

# Import (with progress)
soar db import dump.sql.gz

# Quick query
soar db exec "SHOW TABLES"

# CSV export
soar db csv "SELECT * FROM products" > products.csv
```

### Debugging

```bash
# Check service status
soar ps

# View logs
soar logs
soar logs php

# Interactive shell
soar shell
soar shell nginx

# Run vendor tools
soar bin phpunit
soar bin phpstan
```

## Recent Changes

Based on git history:

1. **Interactive TTY allocation** - Added `-it` flags to preserve colors in Symfony console
2. **Environment file loading order** - Fixed to prevent errors
3. **stopall command** - Stop all running Docker containers
4. **Database import as root** - Improved import reliability
5. **Magento commands** - Added install and clean helpers

## Tips for Claude

When working with soar projects:

1. **Always verify commands** - Check the actual source code in [bin/soar](bin/soar), not just documentation
2. **Service names matter** - Default is `php`, `db`, `nginx` but can be customized via env vars
3. **Environment files** - Look for `.env` and `.env.$APP_ENV` for configuration
4. **Docker Compose pass-through** - Remember that any unknown command goes to docker-compose
5. **Framework detection** - Magento commands only work if `bin/magento` exists and is executable

## Limitations

1. **No built-in multi-environment switching** - Relies on `.env.$APP_ENV` pattern
2. **Assumes docker-compose file structure** - Needs standard service naming
3. **Database commands MySQL-specific** - No PostgreSQL support in shortcuts
4. **No built-in backup rotation** - Manual database backup management
5. **Limited error handling** - Most commands fail silently if services aren't running

## Extending Soar

To add custom commands, edit [bin/soar](bin/soar) and add a new `elif` block:

```bash
elif [ "$1" == "mycommand" ]; then
    shift 1
    $DOCKER_COMPOSE_EXEC -it $DOCKER_PHP_SERVICE <your-command> $@
```

Place it before the final `else` block that passes through to docker-compose.
