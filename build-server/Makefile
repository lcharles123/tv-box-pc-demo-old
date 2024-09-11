NAME=deb-package-builder
IMAGE=lcharles060/$(NAME):latest
CONTAINER=$(NAME)-run

main:
	cat Makefile

dbuild:
	docker build . --tag $(IMAGE)

dpush:
	docker push $(IMAGE)

dpull:
	docker pull $(IMAGE)

drun:
	docker run -td -v $(PWD)/deb_packages:/root --name $(CONTAINER) $(IMAGE) 

denter:
	docker exec -it $(CONTAINER) /bin/bash

dclean:
	docker rm -f $(CONTAINER)

dpurge:
	docker image rm -f $(IMAGE) 
	docker system prune -a

