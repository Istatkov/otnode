# Pruning

Currently on v5 the nodes bid and store all datasets in their databases. With the increased amount of jobs coming, the space grows quite significantly on the node.

There are two levels of pruning:

1. Remove all datasets that have expired on the network - \(L1 pruning\)
2. Low estimated value datasets pruning - Remove all datasets that the node has not won - \(L2 pruning\).

Originally the network was designed to litigate nodes which are not online and pass the contract to another node. However with V6 coming up, this process might be getting a makeover so you can save significant space L2 pruning.

In order to activate them you need to add the following lines to your configuration \(please refer to [Docker installation](../node-installations/docker.md#5-setup-the-configuration-file) for example of the complete configuration file with Pruning enabled\):

```text
   "dataset_pruning": {
      "enabled": true,
      "imported_pruning_delay_in_minutes": 1440,
      "replicated_pruning_delay_in_minutes": 1440,
      "low_estimated_value_datasets": {
        "enabled": true,
        "minimum_free_space_percentage": 30
}
}
```

The setting "true" on the second row activates L1 pruning and "true" on row 5 activates L2 pruning. If you want to disable the pruning just set those two to false. You can also activate L1 pruning and disable L2 pruning.

Also in order for the pruning to work, you need to have deleted any backups from the node inside docker, including the backup directory even if it is empty

```text
docker exec otnode rm -rf ../backup
```



