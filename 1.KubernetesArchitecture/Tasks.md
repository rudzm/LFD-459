1. Create a Kubernetes cluster using the kind tool and the attached configuration file `cluster.yaml`.

    <details>
        <summary>Hint</summary>

        kind create cluster --config <config_file.yaml>
    </details>

2. Create your first Pod using the nginx image and observe its behavior.

    <details>
        <summary>Hint</summary>

        kubectl run <name> --image <image_name>
    </details>

3. Install Cilium CNI can be latest version. And check what changed with Pod.

    <details>
        <summary>Hint</summary>

    - Download `Cillium` binary
      ```bash
      CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)

      CLI_ARCH=amd64

      curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

      sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum

      sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin

      rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
      ```
    - Install CNI
        ```bash
        cilium install
        ```
    </details>

4. Check which node the Pod was created on.

5. Check scheduler logs and create one more Pod.
    <details>
        <summary>Hint</summary>

        kubectl logs <pod_name>
    </details>