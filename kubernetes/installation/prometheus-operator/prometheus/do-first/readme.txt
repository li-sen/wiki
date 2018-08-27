0.additionalScrapeConfigs对prometheus增加额外的配置,key value形式存在
1.如果在prometheus-prometheus.yaml文件配置了additionalScrapeConfigs，一定要先手动生成secret
2.生成secret需要针对每个额外的配置生成
下面是针对名为hadoop-scrape-configs的配置生成的密钥
kubectl delete secret hadoop-scrape-configs -n monitoring
kubectl create secret generic hadoop-scrape-configs --from-file=hadoop-prometheus.yaml -n monitoring


