# Demo app resources for Open Cluster Manager (Advanced Cluster Manager for Kubernetes)

These yaml files are all the individual resources needed to run the vote-app using ACM.

Launch the vote-app using ACM with the following on the hub cluster:

```
oc create -f vote-app.yaml
```

or run:

```
oc create -f https://raw.githubusercontent.com/sjbylo/flask-vote-app/master/open-cluster-management/vote-app/vote-app.yaml 
```

Adjust the placement rule object to ensure the vote-app gets placed into one of the managed clusters. 

To ensure vote-app is deployed to any or all of your managed clusters, change the placement rule spec to:

```
spec:
  clusterSelector:
    matchLabels: {}
```

or use patch to make changes to the placement rule, e.g.

```
oc patch PlacementRule vote-app-placementrule -n vote-app-project --type=json -p '[{"op": "add", "path": "/spec/clusterSelector/matchLabels/environment", "value": "Dev"}]'
oc patch PlacementRule vote-app-placementrule -n vote-app-project --type=json -p '[{"op": "add", "path": "/spec/clusterSelector/matchLabels", "value": {}}]
```

