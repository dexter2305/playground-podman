IMG = docker.io/library/httpd:2.4
PODNAME = "explore-podman-pod"

doc:
	@printf "%s\n" "It does not matter who is publishing the port - the pod or the container but not both, the published port is always accesible to the host.\n"

podman-rootless:
	@printf "%s %s\n" "Podman rootless ? " $$(podman info --format json | jq ".host.security.rootless")

simple-container-publish:
	podman container exists web-cntr-publish || podman run -d --name simple-container --publish 8080:80 $(IMG)
	curl localhost:8080 -i --fail
	printf "Simple container publishing its port is accessible from host.\n"

pod-publish-container:
	podman pod exists $(PODNAME) || podman pod create --name $(PODNAME) --publish 8081:80
	podman run --pod $(PODNAME) --name pod-publishing-container -d --rm $(IMG)
	curl localhost:8081 -i --fail 
	printf "Pod publishing its port is accessible from host.\n"
	
podded-container-publish:
	podman pod exists $(PODNAME) || podman pod create --name $(PODNAME)
	podman container run --name pod-container-publishing -d --publish 8082:80 --rm $(IMG)
	curl localhost:8082 --fail -i
	printf "A container wrapped in a pod. Container publishes its ports is accessible from host.\n"

both-publishing:
	podman pod exists $(PODNAME) || podman pod create --name $(PODNAME) --publish 8083:80
	podman container run --pod $(PODNAME) --name both-publishing -d --publish 8083:80 --rm $(IMG) 

clean:
	podman pod exists $(PODNAME) && podman pod rm -f $(PODNAME) || true
	podman container exists simple-container && podman container rm $$(podman kill simple-container) || true
