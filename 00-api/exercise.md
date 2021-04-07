## List all commands

```
kubectl --help
```

## List the resources 

Show the shortnames and if object is namespaced or not. 

```
kubectl api-resources
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

## List the API versions

```
kubectl api-versions
```

## Display the current config

```
kubectl config view
```

## Display the contexts

```
kubectl config get-contexts
```

