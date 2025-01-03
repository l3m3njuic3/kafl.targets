SHELL := /bin/bash

CUR_DIR := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
ROOT_DIR := $(CUR_DIR)/../

$(ROOT_DIR)/venv/bin/ansible-playbook:
	python3 -m venv $(ROOT_DIR)/venv
	$(ROOT_DIR)/venv/bin/python -m pip install ansible==6.7.0 pywinrm

ansible: $(ROOT_DIR)/venv/bin/ansible-playbook

init:
	vagrant up --no-provision 
	vagrant snapshot save --force 'ready_provision'
	
revoke:
	vagrant snapshot restore 'ready_provision'

gui:
	vagrant up --no-provision
	virt-viewer -a -c qemu:///session vagrant-kafl-windows

compile:
	mkdir -p bin/{userspace,driver}
	x86_64-w64-mingw32-gcc src/userspace/selffuzz_test.c -I ../ -o bin/userspace/selffuzz_test.exe -Wall -mwindows
	x86_64-w64-mingw32-gcc src/driver/vuln_test.c -I ../ -o bin/driver/vuln_test.exe -Wall -lntdll -lpsapi

target: compile ansible
	vagrant up --no-provision
	TARGET_HARNESS='provision_target' vagrant provision
	vagrant halt

no_target: compile ansible
	vagrant up --no-provision --debug
	vagrant provision

start_fuzz:
	@echo "Entering environment in sub-shell"
	source ../../env.sh && \
	kafl fuzz -w work --seed-dir seed --redqueen -p `nproc` --t-soft 255 --purge -i result.json --use_call_stack --t-hard 255

start_fuzzgui:
	@echo "Entering environment in sub-shell"
	source ../../env.sh && \
	kafl gui -w work

setup_init: setup_angr
	mkdir -p ./seed
	mkdir -p bin/driver
	cp ../../fuzzer/Utils/Harness_for_nyx.sys ./bin/driver

setup_angr:
	@echo "Entering environment in sub-shell"
	source ../../env.sh && \
	pip install angr virtualenvwrapper boltons argparse ipdb r2pipe angr-utils && \
	pip install -e angrPT/angr-dev/bingraphvis && \
	pip install -e angrPT/angr-dev/angr-utils

# get IoControlCode aka result.json using angrPT
control_code:
	@echo "Entering environment in sub-shell"
	source ../../env.sh && \
	driver=$$(ls bin/driver | grep -v -e "Harness_for_nyx.sys" -e "vuln_test.exe") && \
	echo "Selected driver: $$driver" && \
	python3 angrPT/angrpt.py -d bin/driver/$$driver && \
	cp result/$${driver%.*}/$${driver%.*}.json result.json && \
	echo "result.json produced successfully"