.PHONY: docs
SHELL := /bin/bash
.DEFAULT_GOAL := help

RED := \u001b[0;31m
NC := \u001b[0m
GREEN := \u001b[0;32m
YELLOW := \u001b[1;33m

define PRINT_HELP_PYSCRIPT
import re, sys

for line in sys.stdin:
    match = re.match(r'^([a-zA-Z_-]+).*?:.*?##(.*)$$', line)
    if match:
        target, help = match.groups()
        print(f'${GREEN}{target :20s}${NC} ${YELLOW}{help}${NC}')
endef
export PRINT_HELP_PYSCRIPT

help:
	@-python -c "$$PRINT_HELP_PYSCRIPT" < $(MAKEFILE_LIST)

submodules:
	@-git submodule init
	@-git submodule sync
	@-git submodule update --remote

start: local-config  ##  start project in containers
	@-docker-compose up -d --build --remove-orphans -t 60 ##--force-recreate

stop:  #  stop project
	@-docker-compose stop
	@-docker-compose down

docker-rmi:
	@-docker rmi -f $(docker images)

docker-rm:
	@-docker stop $(docker ps -a -q)
	@-docker rm $(docker ps -a -q) -f

local-config:  ##  local-config
	@-pipenv run yasha -v local.yml -o docker-compose.yml docker-compose.yml.template

image-build:  ## tworzy obraz dockera
	@-docker build -t $(SERVICE_NAME):latest --rm .

image-upload:  ## wrzuca obraz dockera do rejestru GoogleCloud
	@-docker tag '$(IMAGE_ID)' $(GOOGLE_STORAGE_REGISTRY_PATH)/$(SERVICE_NAME):v$$(sed -ne '/current_version/s/current_version = //p' < setup.cfg)
	@-docker push $(GOOGLE_STORAGE_REGISTRY_PATH)/$(SERVICE_NAME):v$$(sed -ne '/current_version/s/current_version = //p' < setup.cfg)
#	@-docker rmi $(SERVICE_NAME):latest

image-run:
	docker run --name $(SERVICE_NAME) -d -it -p 8000:8000 $(SERVICE_NAME):latest

test:
	echo $(GOOGLE_STORAGE_REGISTRY_PATH)/$(SERVICE_NAME)

%:	%-default
	@	true


push-to-cloud:  ##  wypycha do repo w chmurze
	@-git commit -am 'zmiany'
	bumpversion major
	@-git push origin master --tags

git-pull-all:
	@-git pull
	@-git submodule sync
	@-git submodule update

mykill:  ##  zabija process
	@-if [ "${PROCESS_COMMAND}" = "" ]; then echo 'brak nazwy procesu'; else kill -9 `pgrep -f '$(PROCESS_COMMAND)'`; fi
