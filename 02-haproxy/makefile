.HTTPD = docker.io/library/httpd:2.4
.HAPROXY = docker.io/library/haproxy:alpine3.22
.NAME_PREFIX = proxy
.NETWORK = dev-network

nw: 
	podman network exists $(.NETWORK) || podman network create $(.NETWORK)

deploy: deploy-httpd deploy-router

deploy-httpd: nw
	podman pod exists $(.NAME_PREFIX)-http-server-pod || podman pod create --name $(.NAME_PREFIX)-http-server-pod --infra-name $(.NAME_PREFIX)-http-server-infra \
		--network $(.NETWORK)
	podman container exists $(.NAME_PREFIX)-httpd-server || podman container run --pod $(.NAME_PREFIX)-http-server-pod --name $(.NAME_PREFIX)-http-server \
		--detach $(.HTTPD)

deploy-router: nw
	podman pod exists $(.NAME_PREFIX)-proxy-server-pod || podman pod create --name $(.NAME_PREFIX)-proxy-server-pod --infra-name $(.NAME_PREFIX)-proxy-server-infra \
		--network $(.NETWORK) --publish 8080:8080
	podman container exists $(.NAME_PREFIX)-proxy-server || podman container run --detach --replace --pod $(.NAME_PREFIX)-proxy-server-pod \
		--name $(.NAME_PREFIX)-proxy-server --volume ./haproxy:/usr/local/etc/haproxy:Z $(.HAPROXY)
	podman container exists $(.NAME_PREFIX)-proxy-debugger || podman container run --detach --replace --pod $(.NAME_PREFIX)-proxy-server-pod \
		--name $(.NAME_PREFIX)-proxy-debugger alpine sleep infinity

clean:
	for p in $$(podman pod ls --format "{{.Name}}" --filter name=$(.NAME_PREFIX)); do podman pod rm -f $$p || true; done;
	# podman pod exists $(.PODNAME) && podman pod rm -f $(.PODNAME) || true
	podman network exists $(.NETWORK) && podman network rm $(.NETWORK) || true
