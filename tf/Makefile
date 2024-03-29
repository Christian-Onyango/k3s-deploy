.PHONY: output plan clean help

up: fmt plan apply

help:
	@echo "syntax: make <target>\nWhere target is: ver, debug, clean, output, up, fmt, plan, apply, copy, ip, connect, init, test\n\nUses terraform to manage a simple bastion and a more complex k3s cluster"

debug:
	docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_debug" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   bryandollery/terraform-packer-aws-alpine:14

ver:
	docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_ver" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 -version

fmt:
	time -p docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_fmt" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 fmt -recursive
validate:
	time -p docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_fmt" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 validate

plan: clean
	time -p docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_plan" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 plan -out plan.out

apply: _apply output.json

_apply:
	time -p docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_apply" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 apply plan.out

down:
	time -p docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_down" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 destroy -auto-approve
	rm -f output.json
	rm -f plan.out

test: copy connect

output.json:
	docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_output" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 output -json > output.json

clean:
	rm -f output.json
	rm -f plan.out

copy: output.json
	ssh -oStrictHostKeyChecking=no -i ssh/id_rsa ubuntu@$$(cat output.json | jq '.bastion_ip.value' | xargs) rm -f /home/ubuntu/.ssh/id_rsa
	scp -i ssh/id_rsa ssh/id_rsa ubuntu@$$(cat output.json | jq '.bastion_ip.value' | xargs):/home/ubuntu/.ssh/
	scp -i ssh/id_rsa ssh/id_rsa.pub ubuntu@$$(cat output.json | jq '.bastion_ip.value' | xargs):/home/ubuntu/.ssh/
	ssh -i ssh/id_rsa ubuntu@$$(cat output.json | jq '.bastion_ip.value' | xargs) chmod 400 /home/ubuntu/.ssh/id*

ip:
	echo $$(cat output.json | jq '.bastion_ip.value' | xargs)

connect: output.json
	ssh -oStrictHostKeyChecking=no -i ssh/id_rsa ubuntu@$$(cat output.json | jq '.bastion_ip.value' | xargs)

init: clean
	rm -rf .terraform ssh
	mkdir ssh
	ssh-keygen -t rsa -f ./ssh/id_rsa -q -N ""
	time -p docker container run -it --rm \
		   --env TF_NAMESPACE=$$TF_NAMESPACE \
		   --env AWS_PROFILE="script01" \
		   -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $$PWD:/$$(basename $$PWD) \
		   -v $$PWD/../creds:/.aws \
		   --hostname "$$(basename $$PWD)" \
		   --name "$$(basename $$PWD)_init" \
		   -w /$$(basename $$PWD) \
		   -u $$(id -u):$$(id -g) \
		   --entrypoint terraform \
		   bryandollery/terraform-packer-aws-alpine:14 init

