# Soar

A bash wrapper around Docker Compose that simplifies PHP development workflows.

Soar provides convenient shortcuts for common tasks like running Symfony console commands, Magento CLI, Composer, database operations, and opening shells in containers. Any unrecognized command is passed directly to docker-compose.

## Installation

Add the repository to your global Composer configuration:

```bash
composer global config repositories.soar vcs git@github.com:Roy-Nilsson-AB/soar.git
```

Then install the package:

```bash
composer global require rnab/soar
```

Add Composer's global bin directory to your PATH by adding this to your `~/.bashrc` or `~/.zshrc`:

```bash
export PATH="$HOME/.composer/vendor/bin:$PATH"
```

Then reload your shell or run `source ~/.bashrc`.

## Quick Start

Run soar from your project directory (where your `docker-compose.yml` is located):

```bash
soar up -d              # Start services
soar console cache:clear # Run Symfony command
soar shell              # Open shell in PHP container
soar db                 # Open MySQL client
```

## Commands

| Command | Description |
|---------|-------------|
| `soar up -d` | Start services in background |
| `soar stop` | Stop services |
| `soar restart` | Restart services |
| `soar ps` | Show container status |
| `soar console <cmd>` | Run Symfony console command |
| `soar magento <cmd>` | Run Magento CLI command |
| `soar magento install` | Install Magento with env vars |
| `soar magento clean` | Clear Magento generated files |
| `soar composer <cmd>` | Run Composer |
| `soar bin <tool>` | Run vendor bin tool (phpunit, phpstan, etc.) |
| `soar db` | Open MySQL client |
| `soar db import <file>` | Import SQL dump (supports .gz) |
| `soar db dump` | Export database |
| `soar db exec "<query>"` | Execute SQL query |
| `soar shell [container]` | Open shell in container (default: php) |
| `soar stopall` | Stop ALL Docker containers |
| `soar stopother` | Stop non-project containers |

Any unrecognized command is passed directly to docker-compose.

## Configuration

Soar automatically sources `.env` and `.env.$APP_ENV` from your project directory.

See [.env.example](.env.example) for available configuration options.

### Service Names

Default service names can be overridden via environment variables:

- `DOCKER_PHP_SERVICE` (default: `php`)
- `DOCKER_DB_SERVICE` (default: `db`)
- `DOCKER_WEB_SERVER_SERVICE` (default: `nginx`)

## Documentation

For detailed documentation, see [CLAUDE.md](CLAUDE.md).

## License

MIT

## Author

Roy Nilsson (roy.nilsson@rnab.net)
