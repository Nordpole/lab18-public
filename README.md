# ML/OPS Deployment Infrastructure

Console: https://console-openshift-console.apps-crc.testing/dashboards

## References

   1. CRC 
      [Configuration] (https://access.redhat.com/documentation/en-us/red_hat_openshift_local/2.13/html/getting_started_guide/configuring_gsg)
   2. DB/Storage 
       [OCS] (https://red-hat-storage.github.io/ocs-training/training/ocs4/ocs.html)
       [PostgreSQL from certified image] (https://heidloff.net/article/deploying-postgres-on-openshift/)
       [PostgreSQL on OCS]   (https://cloud.redhat.com/blog/deploy-postgresql-in-openshift-backed-by-openshift-container-storage)
       [PostgreSQL on OCS using Kustomize] (https://stephennimmo.com/deploying-the-red-hat-certified-postgresql-container-on-openshift-using-kustomize-and-sealed-secrets/)
   3. JMS/MQ
       [AMQ Broker] (https://access.redhat.com/documentation/en-us/red_hat_amq/7.7/html-single/deploying_amq_broker_on_openshift/index)



- [x] Databases 
- [~] Openshift Container Storage
- [ ] DBs
    - [x] PostgeSQL
       - [x] [DB and OCS Templates](src/main/ocp/DB/PostgreSQL/README.md)
       - [x] Using Certified Image
       - [x] Using Operator
    - [~] MariaDB
    - [ ] Cassandra

- [x] Cache
- [~] DataGrid/Infinispan
    - [ ] [DataGrid Cluster using Operator](/src/main/ocp/Cache/110_design/005_design.adoc)

1. [x] JMS/MQ
2. [~] Operator for AMQ Broker
3. [ ] AMQ Broker
    - [x] [Using the AMQ Broker Operator](/src/main/ocp/Queue/AMQ/README.md)
    - [~] Using application templates
4. [ ] AMQ Integration
5. [ ] AMQ Streams
6. [ ] Kafka
7. [ ] Pulsar

- [x] FSO
- [~] Openshift Container Storage
    - [ ] S3 Buckets

1. [x] Integration
    - [x]  Camel
    - [ ]  Fuse
    - [ ]  Camel K


- [ ] Stream processing
    - [ ]  Flink
    - [ ]  Spark
    - [ ]  Ignite
