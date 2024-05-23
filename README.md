# openshift_disconnected

Installing OpenShift in a disconnected environment involves several steps. Here’s a general outline:

1. **Prepare the Disconnected Environment:**
   - Ensure that you have a disconnected (air-gapped) network.
         - 
   - Set up a bastion host with internet access to download the required images and content.
   - Set up a local registry in your disconnected environment to host the OpenShift images.

2. **Download OpenShift Installation Files:**
   - On the bastion host, download the OpenShift installer and client binaries from the Red Hat Customer Portal.

3. **Mirror OpenShift Images to Local Registry:**
   - Use the `oc adm release mirror` command to mirror the required OpenShift images to your local registry. This step involves downloading the OpenShift release images and pushing them to your local registry.

   Example:
   ```sh
   oc adm release mirror -a ${PULL_SECRET} \
      --from=quay.io/openshift-release-dev/ocp-release:4.12.0-x86_64 \
      --to=<local_registry_host>:5000/openshift/release \
      --to-release-image=<local_registry_host>:5000/openshift/release:4.12.0-x86_64
   ```

4. **Create Installation Configuration:**
   - Generate the installation configuration files using the `openshift-install create install-config` command. Make sure to specify the local registry in the `install-config.yaml`.

   Example `install-config.yaml`:
   ```yaml
   apiVersion: v1
   baseDomain: example.com
   metadata:
     name: mycluster
   compute:
     - name: worker
       replicas: 3
   controlPlane:
     name: master
     replicas: 3
   networking:
     clusterNetwork:
       - cidr: 10.128.0.0/14
         hostPrefix: 23
     serviceNetwork:
       - 172.30.0.0/16
   platform:
     none: {}
   pullSecret: '{"auths": ... }'
   imageContentSources:
     - mirrors:
         - <local_registry_host>:5000/openshift/release
       source: quay.io/openshift-release-dev/ocp-release
   ```

5. **Install OpenShift:**
   - Use the `openshift-install` binary to create the OpenShift cluster by running `openshift-install create cluster`.

6. **Configure Cluster to Use Local Registry:**
   - Ensure the cluster is configured to use the local registry by updating image contents and image streams.

7. **Post-Installation:**
   - Once the installation is complete, verify that the cluster is running as expected and that it can pull images from the local registry.

Detailed steps, including exact commands and configurations, can be found in the [OpenShift documentation](https://docs.openshift.com/container-platform/latest/installing/installing-restricted-networks-preparations.html).
