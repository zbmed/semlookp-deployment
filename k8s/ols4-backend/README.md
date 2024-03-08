# Install ols4-semlookp version

### Example:
```
helm install ols4 \
--set-json='ingress.dns="ols4.qa.km.k8s.zbmed.de"'  \
--set-json='imageTag="dev"'  \
--set-json='backend.context="/ols4"'  \
--set-json='neo4jTarballUrl="http://ols4-dataserver/neo4j.tgz"'  \
--set-json='solrTarballUrl="http://ols4-dataserver/solr.tgz"'  \
ols4/k8chart/ols4/
```