# PowerScale-CSI-OpenShift-Console

## Youtube video:
- https://youtu.be/ChpasqwvGTo

## Create a new project
- e.g. isilon

## Create isilon-creds secret
- ensure the new project 'isilon' is selected when creating the new secrets
- create a new key/value secret with the secret name = isilon-creds and key = config
- use the isilon-creds.yaml file included in this repo as reference and paste into the text field
- edit with your environment details (e.g. endpoint IP address, username, password)

## Create isilon-certs secret
- create a new secret from yaml
- use the isilon-certs.yaml file included in this repo and replace the default yaml template
- no further changes are required

## Install Dell CSI Operator using Operator Hub
- use default options
- stable, all namespaces, namespace = openshift-operators, automatic approval
- view operator once complete and confirm status 'succeeded'

## Create CSIIsilon instance
- ensure 'isilon' project is selected in the drop down
- go to CSI PowerScale and create CSIIsilon instance
- Add the following field values;
-- Name: isilon
-- Auth Secret: isilon-creds
-- TLSCert Secret: isilon-certs-0
-- Common specifications --> Container Environment vars --> X_CSI_ISI_AUTH_TYPE: 1

## After deployment of the instance, check the pods are running in the namespace
```
NAME                                READY   STATUS    RESTARTS   AGE
isilon-controller-f8d645c48-5tqhp   5/5     Running   0          65m
isilon-controller-f8d645c48-xkhkx   5/5     Running   0          65m
isilon-node-8jtpj                   2/2     Running   0          65m
isilon-node-b489f                   2/2     Running   0          65m
isilon-node-vdhtz                   2/2     Running   0          65m
```

## Create storage class
- Name: csi-isilon-sc
- Reclaim policy: Delete
- Volume binding mode: Immediate
- Provisioner: csi-isilon.dellemc.com
- Allow PersistentVolumeClaims to be expanded

#############################################
# Test the CSI driver
#############################################

# Create a new project
- e.g. test

# Create a new PersistentVolumeClaim
- StorageClass: csi-isilon-sc
- PersistentVolumeClaim name: pvol0
- Access mode: Single user (RWO)
- Size: 1GiB
- Volume mode: Filesystem

# Create a test pod
- use the test_pod.yaml included in this repo

# Log into the pod and test the volume
- check the mount
```
mount | grep data0
```
- check the path it was mounted to
```
cd data0
df -h .
ls -al
```
- check for read / write access
```
touch test
vi test
```
- check the file size
```
ls -al
```

# Create a volumesnapshot class
- use the isilon-volumesnapshotclass-v1.yaml included in this repo

# Create a volume VolumeSnapshot
- select the pvol0 pvc created earlier
- give it a name
- select the isilon-snapclass created earlier

# Check the snapshot was created successfully

### Login to the PowerScale and confirm the volume / snapshot
