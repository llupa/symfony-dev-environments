# Symfony Development Environment - Makefile Template
# ====================================================
# Common automation commands for Symfony projects.
# Copy this file to your project root and customize as needed.
#
# Usage:
#   make install    # First-time setup
#   make run        # Start development environment
#   make stop       # Stop all services
#   make help       # Show all available commands

# =============================================================================
# Configuration
# =============================================================================

# PHP version (should match your Brewfile)
PHP_VERSION ?= 8.5

# Docker Compose command (docker-compose or docker compose)
DOCKER_COMPOSE ?= docker compose

# Symfony console command
SYMFONY_CONSOLE ?= symfony console

# Web server port
SERVER_PORT ?= 8000

# Default target
.DEFAULT_GOAL := help

# =============================================================================
# Installation
# =============================================================================

.PHONY: install
install: ## Install all dependencies and set up the environment
	@echo "Installing Homebrew dependencies..."
	brew bundle install --verbose
	@echo ""
	@if [ "$(uname)" == "Darwin" ]; then \
		sudo codesign --force --deep --sign - $(readlink -f `which symfony`); \
	fi
	@echo ""
	@echo "Setting up Symfony proxy..."
	symfony server:ca:install
	symfony proxy:start
	@echo ""
	@echo "Installing Composer dependencies..."
	composer install
	@echo "Installation complete!"
	@echo "Run 'make run' to start the development environment."

.PHONY: update
update: ## Update all dependencies
	brew update && brew upgrade
	composer update

# =============================================================================
# Development Server
# =============================================================================

.PHONY: run
run: docker-up server-start ## Start Docker and Symfony server

.PHONY: stop
stop: server-stop docker-down ## Stop Symfony server and Docker

.PHONY: restart
restart: stop run ## Restart the development environment

.PHONY: server-start
server-start: ## Start the Symfony local web server
	symfony serve --port=$(SERVER_PORT) --daemon
	@echo ""
	@echo "Server running at https://127.0.0.1:$(SERVER_PORT)"

.PHONY: server-stop
server-stop: ## Stop the Symfony local web server
	symfony server:stop || true

.PHONY: server-log
server-log: ## Show Symfony server logs
	symfony server:log

# =============================================================================
# Docker
# =============================================================================

.PHONY: docker-up
docker-up: ## Start Docker containers
	$(DOCKER_COMPOSE) up -d

.PHONY: docker-down
docker-down: ## Stop Docker containers
	$(DOCKER_COMPOSE) down

.PHONY: docker-restart
docker-restart: docker-down docker-up ## Restart Docker containers

.PHONY: docker-logs
docker-logs: ## Show Docker container logs
	$(DOCKER_COMPOSE) logs -f

.PHONY: docker-ps
docker-ps: ## List running Docker containers
	$(DOCKER_COMPOSE) ps

.PHONY: docker-clean
docker-clean: ## Remove all Docker containers, volumes, and images for this project
	$(DOCKER_COMPOSE) down -v --rmi local

# =============================================================================
# Database
# =============================================================================

.PHONY: db-create
db-create: ## Create the database
	$(SYMFONY_CONSOLE) doctrine:database:create --if-not-exists

.PHONY: db-drop
db-drop: ## Drop the database
	$(SYMFONY_CONSOLE) doctrine:database:drop --force --if-exists

.PHONY: db-migrate
db-migrate: ## Run database migrations
	$(SYMFONY_CONSOLE) doctrine:migrations:migrate --no-interaction

.PHONY: db-diff
db-diff: ## Generate a migration by comparing entities to database
	$(SYMFONY_CONSOLE) doctrine:migrations:diff

.PHONY: db-reset
db-reset: db-drop db-create db-migrate ## Reset the database (drop, create, migrate)

.PHONY: db-fixtures
db-fixtures: ## Load data fixtures
	$(SYMFONY_CONSOLE) doctrine:fixtures:load --no-interaction

# =============================================================================
# Cache
# =============================================================================

.PHONY: cache-clear
cache-clear: ## Clear the Symfony cache
	$(SYMFONY_CONSOLE) cache:clear

.PHONY: cache-warmup
cache-warmup: ## Warm up the Symfony cache
	$(SYMFONY_CONSOLE) cache:warmup

# =============================================================================
# Code Quality
# =============================================================================

.PHONY: lint
lint: ## Run all linters
	$(SYMFONY_CONSOLE) lint:yaml config
	$(SYMFONY_CONSOLE) lint:twig templates
	$(SYMFONY_CONSOLE) lint:container

.PHONY: cs-fix
cs-fix: ## Fix code style issues (requires PHP-CS-Fixer)
	vendor/bin/php-cs-fixer fix

.PHONY: phpstan
phpstan: ## Run PHPStan static analysis (requires PHPStan)
	vendor/bin/phpstan analyse

# =============================================================================
# Testing
# =============================================================================

.PHONY: test
test: ## Run the test suite
	php bin/phpunit

.PHONY: test-coverage
test-coverage: ## Run tests with coverage report
	XDEBUG_MODE=coverage php bin/phpunit --coverage-html var/coverage

# =============================================================================
# Symfony Proxy
# =============================================================================

.PHONY: proxy-start
proxy-start: ## Start the Symfony proxy
	symfony proxy:start

.PHONY: proxy-stop
proxy-stop: ## Stop the Symfony proxy
	symfony proxy:stop

.PHONY: proxy-status
proxy-status: ## Show proxy status and attached domains
	symfony proxy:status

.PHONY: proxy-attach
proxy-attach: ## Attach current directory to proxy (usage: make proxy-attach DOMAIN=myapp)
	@if [ -z "$(DOMAIN)" ]; then echo "Usage: make proxy-attach DOMAIN=myapp"; exit 1; fi
	symfony proxy:domain:attach $(DOMAIN)
	@echo "Project attached to https://$(DOMAIN).wip"

# =============================================================================
# Composer
# =============================================================================

.PHONY: composer-install
composer-install: ## Install Composer dependencies
	composer install

.PHONY: composer-update
composer-update: ## Update Composer dependencies
	composer update

.PHONY: composer-require
composer-require: ## Require a package (usage: make composer-require PACKAGE=vendor/package)
	@if [ -z "$(PACKAGE)" ]; then echo "Usage: make composer-require PACKAGE=vendor/package"; exit 1; fi
	composer require $(PACKAGE)

# =============================================================================
# Help
# =============================================================================

.PHONY: help
help: ## Show this help message
	@echo "Symfony Development Environment"
	@echo ""
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'
