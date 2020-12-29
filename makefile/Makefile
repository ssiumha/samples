.DEFAULT_GOAL := help

help:
	@awk 'BEGIN { FS="##" } \
		match($$1, /^([0-9A-Za-z%_-]+):([^=]|$$| ##)/, a) { \
			printf "\033[36m%-32s\033[0m %s\n", a[1], $$2 \
		}' $(MAKEFILE_LIST) \
		| sort

# ================================================================
# makefile options
SHELL := /usr/bin/env bash


# ================================================================
# misc statement

# * include로 파일 분할 가능, 상위 파일에서 선언된 모든 변수 정보를 유지함
# * 같은 target 정의가 불가능
# * template 마냥 붙여넣기 된다고 느끼면 될듯?
# TODO
# include scripts/sub.mk

# 매 명령마다 새 프로세스가 뜨므로 상태를 유지하려면
# ()나 {}로 감싸거나 파이프라인으로 한줄명령을 만들어야한다
test-state:
	TEST_STATE=1;
	echo $$TEST_STATE;	# print>
	( \
		TEST_STATE=2; \
		echo $$TEST_STATE; \
	) # print> 2


# ================================================================
# variable vs env
# * case sensitive
# * 입력받은 변수가 없을 때만 입력하려면 `?=`
# * `=`는 변수가 사용될 떄 우측항을 재귀적으로 평가, `:=`는 비재귀적으로 선언시 평가된다
#   * ex) `OPT = $(OPT) -o`는 무한 루프에 빠져 에러를 발생시킨다
# * 환경변수로 입력받을 때와 make의 변수인자로 받을 때 동작이 다르다
# 	* 환경변수로 받았을 때 `?=`로 선언하지 않았다면 덮어쓴다
# 	* 변수인자로 받으면 기존 변수 선언이 평가조차 되지 않고 무시된다

# make test-variable          	# output> variable /
# AAA=123 make test-variable  	# output> variable / variable			* ?= 로 선언되었다면 123 / 123
# make test-variable AAA=123  	# output> 123 / 123
AAA = variable
test-variable:
	echo $(AAA) / $$AAA
test-variable-all:
	make test-variable
	@echo '---'
	AAA=123 make test-variable
	@echo '---'
	make test-variable AAA=123

# make test-env 					 	# output> /
# BBB=123 make test-env    	# output> 123 / 123
# make test-env BBB=123 		# output> 123 / 123
test-env:
	echo $(BBB) / $$BBB
test-env-all:
	make test-env
	@echo '---'
	BBB=123 make test-env
	@echo '---'
	make test-env BBB=123


# ================================================================
# use case: %
deploy-stg: DEPLOY_TARGET = staging	## deploy staging
deploy-prod: DEPLOY_TARGET = production ## deploy production
deploy-%:
	echo deploy-$(DEPLOY_TARGET)

# 직교성을 활용할 수 있다
# * help로 보기쉽게 안걸리는게 단점
# * help가 필요하다면 `watch-stg: ##` 형태로 선언해두어야함
%-stg: ENV_TARGET = staging
%-prod: ENV_TARGET = production
watch-%: ## watch-stg watch-prod
	watch echo $(ENV_TARGET)
logs-%:	## logs-stg logs-prod
	echo tail -f $(ENV_TARGET)


# ================================================================
# boilerplate command
BOILERPLATE_COMMAND = docker exec -it blabla
boiler-cmd1:
	echo $(BOILERPLATE_COMMAND) bash

boiler-cmd2:
	echo $(BOILERPLATE_COMMAND) bundle install

boiler-cmd3:
	echo $(BOILERPLATE_COMMAND) rails console
