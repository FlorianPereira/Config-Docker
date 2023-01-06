DOCKER_COMPOSE?=docker-compose
RUN=$(DOCKER_COMPOSE) run --rm php
EXEC?=$(DOCKER_COMPOSE) exec php
COMPOSER=$(DOCKER_COMPOSE) exec -u deploy php composer
CONSOLE=$(EXEC) bin/console
PHPUNIT=$(EXEC) vendor/bin/phpunit --configuration //var/www/html/phpunit.xml
PHPMD?=$(EXEC) vendor/bin/phpmd
PHPUNIT_ARGS?=-v
DOCKER_FILES=$(shell find ./images -type f -name '*')

.DEFAULT_GOAL := help
.PHONY: help start up stop reset clean test tu
.PHONY: build up deps phpcs phpcsfix tty deploy-prepare
.PHONY: rm-docker-dev.lock

help:
	@grep -E '(^[a-zA-Z_-]+:.*?##.*$$)|(^##)' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[32m%-30s\033[0m %s\n", $$1, $$2}' | sed -e 's/\[32m##/[33m/'

##
## Project run
##---------------------------------------------------------------------------

up: ## Run project
	$(DOCKER_COMPOSE) up -d --remove-orphans

##
## Project setup
##---------------------------------------------------------------------------

start: create-network build up  ## Install and start the project

stop:                                                                                                  ## Remove docker containers
	$(DOCKER_COMPOSE) kill
	$(DOCKER_COMPOSE) rm -v --force

reset: stop start

clear:                                                                                            ## Remove all the cache, the logs, the sessions and the built assets

clean: clear                                                                                           ## Clear and remove dependencies
	rm -rf vendor

tty:                                                                                                   ## Run app container in interactive mode
	$(RUN) /bin/sh

##
## Database
##---------------------------------------------------------------------------

db: vendor                                                                               ## SetUp DynamoDB
	$(RUN) php /var/www/html/tests/script/create_table_if_not_exists.php

ssm: vendor ## SetUp SSM in local stack and put secure item
	$(RUN) php /var/www/html/tests/script/init_local_ssm.php

##
## Tests
##---------------------------------------------------------------------------

test: tu phpcs                                                                                    ## Run the PHP Unit and newman test

test-phpunit:                                                                                          ## Run phpunit tests
	$(PHPUNIT) $(PHPUNIT_ARGS)

tu: vendor                                                             ## Run the PHP unit tests
	$(PHPUNIT)

tu-coverage: vendor                                                             ## Run the PHP unit tests
	$(PHPUNIT) --coverage-html ./coverage                                                          ## Run the PHP functional tests

phpmd: vendor   ## Launching PHP Mess Detector (cleancode,codesize,design,naming,unusedcode)...
	$(PHPMD) ./app/ text cleancode,codesize,design,naming,unusedcode  --exclude '/var/www/html/app/TreezorAPI/'

phpcs: vendor                                                                                          ## Lint PHP code
	$(EXEC) /var/www/html/vendor/bin/phpcs --standard=PSR1,PSR2 --ignore=/var/www/html/app/TreezorAPI/* /var/www/html/app/

phpcsfix: vendor                                                                                       ## Lint and fix PHP code to follow the convention
	$(EXEC) /var/www/html/vendor/bin/phpcbf --ignore=/var/www/html/app/TreezorAPI/* --standard=PSR1,PSR2 /var/www/html/app/

##
## Dependencies
##---------------------------------------------------------------------------

deps: vendor                                                                                ## Install the project PHP

##
## Deploy
##---------------------------------------------------------------------------

deploy-prepare: ## Prepare deploy, install prod deps and optimize
	rm -rf vendor
	$(COMPOSER) install --prefer-dist --optimize-autoloader --no-dev

##


# Internal rules

build: docker-dev.lock

create-network:
	docker network create getstarted2 || true
docker-dev.lock: $(DOCKER_FILES)
	$(DOCKER_COMPOSE) pull --ignore-pull-failures
	$(DOCKER_COMPOSE) build --force-rm --pull

# Rules from files

vendor: composer.lock
	$(COMPOSER) install -n

composer.lock: composer.json
	@echo composer.lock is not up to date.