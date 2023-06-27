# First install
First install wont run because of the following parameters:

````yaml
  gitlab:
    ...
    migrations:
      annotations:
        "helm.sh/hook": "pre-install"
        "helm.sh/hook-delete-policy": "before-hook-creation,hook-succeeded"
````

You have to remove those for the first sync.

# Restore Gitlab from prod backup

### Get latest backup:
    
    $ export AWS_PROFILE=randstad-prod
    $ aws s3 ls gitlab-backups-service | sort | tail -n 1 | cut -d " " -f 5 | sed 's/_gitlab_backup.tar//'
    > 1686912418_2023_06_16_15.4.6-ee

The timestamp to extract is: `1686912418_2023_06_16_15.4.6-ee`

### Restore

    $ kubie ctx sandbox

    $ kubectl exec -it $(kubectl get po -l app=toolbox -n gitlab -o=jsonpath='{.items[].metadata.name}') -n gitlab -it -- env BACKUP_BUCKET_NAME=gitlab-backups-service backup-utility --restore -t 1686923491_2023_06_16_15.4.6-ee


### Connect to toolbox to disable ci_pipeline_schedules

    $ kubectl get secret/gitlab-secret -n gitlab --template='{{index .data "postgresql-password"}}' | base64 -D
    $ kubectl exec -it $(kubectl get po -l app=toolbox -n gitlab -o=jsonpath='{.items[].metadata.name}') -n gitlab -it -- gitlab-rails dbconsole
    # Wait for prompt... and enter password

    $ UPDATE ci_pipeline_schedules SET active='f';

# Upgrading gitlab
Upgrading gitlab is a tricky because ArgoCD is plugged on it.
Even Gitlab itself is sync via ArgoCD and via itself.

So you need to:

## 1. Make a backup of gitlab
    
    $ kubie ctx service
    $ kubectl exec -it $(kubectl get po -l app=toolbox -n gitlab -o=jsonpath='{.items[].metadata.name}') -n gitlab -it -- backup-utility --skip registry --skip artifacts

## 2. Push you code on another repository
TODO

# Upgrading 6.11.x to 7.xx
(from: https://docs.gitlab.com/charts/installation/upgrade.html#update-the-bundled-redis-sub-chart )

    $ kubectl scale deployment --replicas 0 --selector 'app in (webservice, sidekiq, kas, gitlab-exporter)' --namespace gitlab
    $ kubectl delete statefulset gitlab-redis-master --namespace gitlab

    // Go sync

    $ kubectl scale deployment --replicas 2 --selector 'app in (webservice, sidekiq, kas)' --namespace gitlab
    $ kubectl scale deployment --replicas 1 --selector 'app in (gitlab-exporter)' --namespace gitlab
