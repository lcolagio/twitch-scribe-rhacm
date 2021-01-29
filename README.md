# Scribe, Submariner, and ACM
Two clusters have already been created before beginning this scenario. The primary cluster has also been added into ACM.

Perform all of the steps with the KUBECONFIG for the ACM server exported.

## Deploying our Application
We will deploy a Dokuwiki application. A HAProxy load balancer is used to pass traffic between primary and failover clusters.
```
oc create -f acm-app-configuration
```

## Scribe components
Deploy scribe components first. Scribe is installed on all OpenShift clusters.
```
oc create -f acm-scribe-deployment-configuration
```

## Failover cluster import
From the UI import the failover cluster. Add the labels of site=failover and purpose=dokuwiki.

## Replication
Scribe will automatically be installed. We need to define the replication destination.
```
oc create -f acm-replication-configuration/scribe-rsync-failover-acm-configuration/
```

## Git
Copy the secret and the clusterIP. Scrub out all of the extra metadata.
```
vi rsync-replication/source-rsync/scribe-rsync-dest-src-database-destination.yaml
oc get secret scribe-rsync-dest-src-database-destination -n dokuwiki -o yaml > rsync-replication/source-rsync/scribe-rsync-dest-src-database-destination.yaml
```

## Create the replication source
We need to define the replication source.
```
oc create -f acm-replication-configuration/scribe-rsync-primary-acm-configuration/
```

## Verify replication
Export the kubeconfig of the failover cluster and show volumesnapshots.
```
oc get volumesnapshot -n dokuwiki
```

## Remove label
Remove the label purpose=dokuwiki from the primary cluster. This will cause the application to failover to the other cluster
