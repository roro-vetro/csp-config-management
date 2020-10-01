# Redis Application Package

This is a [kpt](https://github.com/GoogleContainerTools/kpt) package to deploy a
Redis cluster managed by
[Ucloud Redis Operator](https://github.com/ucloud/redis-operator).

## Installation

First ensure you have an ACM git repository with `sourceFormat: unstructured`.
Assume the repository is checked out to `~/acm` on your workstation.

Get the package with `kpt`.

```shell
$ kpt pkg get sso://team/abox-team/abox.git/dev/demo/redis/application ~/acm/redis
$ cd ~/acm/redis
```

### [Optional] Customize Parameters

You can set customizable parameters in the package. Firstly, display the setters
that are provided to the package.

```
$ kpt cfg list-setters .
  NAME         VALUE     SET BY            DESCRIPTION             COUNT   REQUIRED
cluster-name    redis               Redis cluster name               1       No
namespace       redis                                                7       No
...
```

Then, change the parameters with `kpt cfg set . <param-name> <value>`. For
example:

```shell
$ kpt cfg set . cluster-name redis
```

### Deploy

Commit the newly added files and push to your ACM repository.

```shell
$ git add .
$ git commit -m "Add redis application"
$ git push
```

This will create the following things in your cluster:

1.  A Redis Operator.
2.  A Redis cluster managed by the Redis operator.

## Visibility

You can install
[RedisInsight](https://redislabs.com/redis-enterprise/redis-insight/) by
committing **redisinsight/redisinsight.yaml** to ACM repository. This YAML file
creates a RedisInsight deployment in your cluster, then you can use exposed port
to set up your RedisInsight.

If you prefer to use other Web UI, just remove
**redisinsight/redisinsight.yaml** and commit to remote.

## Scaling

1.  To scale the cluster, use kpt to change the **replica-size** parameter.

    ```shell
    $ kpt cfg set . replica-size 5 # Scale the cluster to have 5 replicas
    ```

2.  Commit the modified file to ACM repository.

## Reconfiguration

1.  To change non-scaling parameters for Redis(e.g. parameters in redis.conf),
    edit the **redis-cluster.yaml** and add parameter:value pair under `config`
    spec.

    For example:

    ```shell
    spec:
      config:
        hz: "12"
        loglevel: debug
        maxclients: "10000"
    ```

2.  Commit the modified **redis-cluster.yaml** file to ACM repository.

## Security Features

### Password

You can edit `spec.password` field in **redis-cluster.yaml** to create a
[AUTH](https://redis.io/commands/auth) password for Redis. In order to disable
authentication, leave the `spec.password` field empty.

NOTE: For a Redis 6.0 instance, or greater, is using the
[Redis ACL system](https://redis.io/topics/acl). When `spec.password` is set,
the username for this password is `default`.

## Uninstallation

1.  Remove the application directory(redis in this example) in ACM git
    repository.
2.  Commit the change to remote.

NOTE: Removing all namespaces or cluster-scoped resources in a single commit
leads to
[Config Manangement Errors](https://cloud.google.com/anthos-config-management/docs/reference/errors#knv2006).

