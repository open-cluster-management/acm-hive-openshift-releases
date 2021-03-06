# NOTICE:
On **12-07-2020** we changed the default naming convention for the images. We added a `-appsub` suffix, so that we would not block upgrades nor easily overlap with user created images. If you directly reference an image name in your scripts, you will need to modify your image reference to include the `-appsub` suffix.

# OpenShift Release Images
This repository provides a subscription that will populate all the latest OpenShift images into Advanced Cluster Management for OpenShift deployments. More information about OpenShift release images and the channels mentioned below can be found here: https://docs.openshift.com/container-platform/4.4/updating/updating-cluster-between-minor.html#understanding-upgrade-channels_updating-cluster-between-minor

# Repository layout
- `fast` `./clusterImageSets/fast/*` : The fast channel OpenShift releases, generally available and fully supported
- `stable` `./clusterImageSets/stable/*` : The latest stable OpenShift releases, same as fast, but with connected customer feedback
- `candidate` `./clusterImageSets/releases/*` : All release images are in this folder, those labelled `candidate` releases are not supported

See the `Custom curated` section on controlling your own OpenShift release timelines with Advanced Cluster Management

## Latest supported images (ONLINE)
- Populates the latest OpenShift `fast` release images
- Run the following command
```bash
# Connect to you Red hat Advanced Cluster Management hub
# On OpenShift Dedicated, run "oc new-project hive-clusterimagesets"
make subscribe-fast
```
- After about 60s the Create Cluster console will list the latest supported OpenShift images
### Stable channel images
- Populates the latest 2x `stable` release images
```bash
make subscribe-stable
```

### Alternate full CLI command
- Populates the latest `fast` release images
```bash
oc apply -k subscribe/
```

### How to pause the subscription
#### Prerequisites
1. The fast channel subscription has been applied
2. Logged into the ACM hub
#### Pause the Fast Channel subscription
```bash
make pause-fast

# Full CLI command:
oc -n hive patch appsub hive-clusterimagesets-subscription-fast-0 --type='json' -p='[{"op":"replace","path": "/metadata/labels/subscription-pause","value":"true"}]'
```
#### Unpause the Fast Channel subscription
```bash
make unpause-fast

# Full CLI command:
oc -n hive patch appsub hive-clusterimagesets-subscription-fast-0 --type='json' -p='[{"op":"replace","path": "/metadata/labels/subscription-pause","value":"false"}]'
```
_Note: `hive-clusterimagesets-subscription-stable-0` resource name can be substituted, in the CLI pause commands, if you are working with `stable` release images._

### Continuous updates
- This repository periodically updates as new fast and stable release images are minted
- Changes in this repository will be applied to your subscribed cluster
- This is the stable channel list being followed by this repository, [link](https://github.com/openshift/cincinnati-graph-data/blob/master/channels/stable-4.3.yaml)

### Uninstall
```bash
make unsubscribe-all

# Full CLI commands:
oc delete -k subscribe/
oc delete -f subscribe/subscription-stable.yaml  #If your using the stable channel
```

## Custom curated (ONLINE)
- Fork this repository
- Update the the `./subscribe/channel.yaml` file, changing the organization `open-cluster-management` to your `organization_name` or `github_username` where you forked the repository.
```yaml
spec:
  type: GitHub
  pathname: https://github.com/NAME_or_ORGANIZATION/acm-hive-openshift-versions.git
```
- Place the YAML files for the images you want to appear in the Red Hat Advanced Cluster Management console under `./clusterImageSets/stable/*` or `./clusterImageSets/fast/*`
- Commit and push your changes to the forked repository
- Run the following command
```bash
make subscribe-fast   #fast channel
make subscribe-stable #stable channel
```
- After about 60s the Create Cluster console will list the new images available from your forked repository
- Add new OpenShift install images by created additional files in the `clusterImageSets/stable/` or `clusterImageSets/fast/` directories

## How to get new versions
- This repository will automatically update with the latest stable and fast versions
- You can monitor this repository and merge changes to your forked repository
- As soon as new images are committed to this repository and merged to your fork, they will become available in the Red Hat Advanced Cluster Management console (about 60s)
- This is the install image repository being used: https://quay.io/repository/openshift-release-dev/ocp-release?tab=tags
- These are the streams being followed by this repository: https://github.com/openshift/cincinnati-graph-data/blob/master/channels/

## (Technical Preview) Usecase - OFFLINE - limited images
- Copy the `clusterImageSets` directory to a system that has access to the disconnected Red Hat Advanced Cluster Management Hub
- Delete the YAML files for OpenShift versions you do not want to host OFFLINE
- Modify the `clusterImageSet` YAML files for the remaining OpenShift release images to point to the correct `OFFLINE` repository
```yaml
apiVersion: hive.openshift.io/v1
kind: ClusterImageSet
metadata:
    name: img4.4.0-rc.6-x86-64
spec:
    releaseImage: IMAGE_REGISTRY_IPADDRESS_or_DNSNAME/REPO_PATH/ocp-release:4.4.0-rc.6-x86_64
```
- Make sure the images are loaded in the OFFLINE image registry referenced in the YAML
- Apply a subset of the YAML files
```bash
oc apply -f clusterImageSets/CHANNEL/VERSION/FILE_NAME.yaml
```
- The Create Cluster console will list only the images available from the `cluseterImageSets` directory

## Secure Github repostiory
- Uncomment the secret reference in `subscribe/channel.yaml`
- Create a `subscribe/secret.yaml` file with the following contents
```yaml
  ---
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-github-secret
    namespace: ocp-clusterimagesets
  data:
    user: BASE64_ENCODED_GITHUB_USERNAME
    accessToken: BASE64_ENCODED_GITHUB_TOKEN
```
- The following command is used to encode base64: `echo "VALUE_TO_ENCODE" | base64`  place the output in the yaml file.
- Create the secret
```bash
make subscribe-fast
# OR
make subscribe-stable
```

# Development, to support a new OpenShift version via Travis
Update the Travis variable: `LIST_VERSIONS`
Make sure to enclose the space separated version numbers with quotes.
```
LIST_VERSIONS = "4.3 4.4 4.5"
```