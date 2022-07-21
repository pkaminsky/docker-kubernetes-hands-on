# Docker registry

You can pull things from a registry with `docker pull` or push them.

The way this works is based on the `tags` we applied before.

When you push them, the `tag` of the image comprises a URL to that image in a docker registry.

registry/team/containername:latest

so you could tag your image

> docker push regsitry/team/containername:latest

And the tag is not only the name of the image in the registry, but also the path to the registry.