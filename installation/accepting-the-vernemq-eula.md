# Accepting the VerneMQ EULA

To use the VerneMQ pre-built packages and Docker images you have to accept the [VerneMQ EULA](https://vernemq.com/end-user-license-agreement). Make sure to read and understand the EULA before accepting it.


## For Docker Images

For Docker images the EULA can be accepted by setting the environment variable`DOCKER_VERNEMQ_ACCEPT_EULA=yes`, for Docker Swarm add `DOCKER_VERNEMQ_ACCEPT_EULA: yes` to the environment.

For the Helm chart the EULA for the Docker images can be accepted by extending the `additionalEnv` section with:

`additionalEnv:   
    - name: DOCKER_VERNEMQ_ACCEPT_EULA   
      value: "yes"`

and similarly for the [VerneMQ Operator](../guides/vernemq-on-kubernetes.md#deploy-vernemq-using-the-kubernetes-operator), to accept the EULA for the Docker images, the `env` can be extended with:

`env:   
    - name: DOCKER_VERNEMQ_ACCEPT_EULA   
      value: "yes"`

