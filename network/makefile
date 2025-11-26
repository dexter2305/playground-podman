.NAME_PREFIX = network
.NETWORK_NAME = $(.NAME_PREFIX)-dev-network
.PODNAME = cx-network-pod
.IMG = docker.io/library/httpd:2.4

doc: 
	printf "%s\n" "--network connects the resource (pod/container) to the network."
	printf "%s\n" "When a container joins the pod, container inherits the pod's network namespace."
	printf "%s\n" "For interpod communication, use the internal port of server, given that both pods are on the same network."

pod-rootless:
	@printf "%s %s\n" "Podman rootless ? " $$(podman info --format json | jq ".host.security.rootless")

clean:
	for c in $$(podman container ls --format "{{.Names}}" --filter name=$(.NAME_PREFIX)); do podman container rm $$(podman kill $$c) || true; done;
	for p in $$(podman pod ls --format "{{.Name}}" --filter name=$(.NAME_PREFIX)); do podman pod rm $$p || true; done;
	# podman pod exists $(.PODNAME) && podman pod rm -f $(.PODNAME) || true
	podman network exists $(.NETWORK_NAME) && podman network rm $(.NETWORK_NAME) || true

nw:
	podman network exists $(.NETWORK_NAME) || podman network create $(.NETWORK_NAME)

simple-container-publish: nw
	podman container exists $(.NAME_PREFIX)-simple-container || podman run -d --name $(.NAME_PREFIX)-simple-container --publish 8080:80 --network $(.NETWORK_NAME) $(.IMG)
	curl localhost:8080 -i --fail
	@printf "Simple container publishing its port is accessible from host.\n"

pod-publish-container:
	podman pod exists $(.PODNAME) || podman pod create --name $(.PODNAME) --publish 8081:80 --network $(.NETWORK_NAME)
	podman container run --pod $(.PODNAME) --name $(.NAME_PREFIX)-pod-publishing-container -d --rm $(.IMG)
	curl localhost:8081 -i --fail 
	@printf "Pod publishing its port is accessible from host.\n"

podded-container-publish:
	podman pod exists $(.PODNAME) || podman pod create --name $(.PODNAME)
	podman container run --pod $(.PODNAME) --name $(.NAME_PREFIX)-pod-container-publishing -d --publish 8082:80 --rm --network $(.NETWORK_NAME) $(.IMG)
	curl localhost:8082 --fail -i
	@printf "A container wrapped in a pod. Container publishes its ports is accessible from host.\n"

both-networking:
	podman pod exists $(.PODNAME) || podman pod create --name $(.PODNAME) --publish 8083:80 --network $(.NETWORK_NAME)
	podman container run --pod $(.PODNAME) --name $(.NAME_PREFIX)-both-publishing -d --publish 8083:80 --rm --network $(.NETWORK_NAME) $(.IMG) 
	curl localhost:8083 -i
	@printf "%s\n" "When a container joins a pod, it inherits pods network namespace. --network at the container level is redundant"

pod-network-container-publishes:
	podman pod exists $(.PODNAME) || podman pod create --name $(.PODNAME) --network $(.NETWORK_NAME)
	podman container run --pod $(.PODNAME) --name  $(.NAME_PREFIX)-pod-network-container-publishes -d --publish 8084:80 --rm --network $(.NETWORK_NAME) $(.IMG) 
	curl localhost:8084 -i
	@printf "%s\n" "When a container joins a pod, it inherits pods network namespace. --network at the container level is redundant"

interpod-comm: nw
	podman pod exists $(.NAME_PREFIX)-server-pod || podman pod create --name $(.NAME_PREFIX)-server-pod --network $(.NETWORK_NAME) --publish 8085:80 
	podman pod exists $(.NAME_PREFIX)-client-pod || podman pod create --name $(.NAME_PREFIX)-client-pod --network	$(.NETWORK_NAME)
	podman container exists $(.NAME_PREFIX)-server || podman container run --name $(.NAME_PREFIX)-server --pod $(.NAME_PREFIX)-server-pod -d --rm $(.IMG)
	podman container run --replace --name $(.NAME_PREFIX)-client --pod $(.NAME_PREFIX)-client-pod --rm alpine sh -c "apk add iputils-ping curl; curl -s $(.NAME_PREFIX)-server-pod:80 -i"
	curl http://localhost:8085 -s -i
	printf "%s\n" "Interpod communication requires two or more pods to be on the same network and client container must connect to the server's internal port."
	printf "%s\n" "Published port is only to be accessible from the host machine"
