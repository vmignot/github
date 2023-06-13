# Restore Gitlab from prod backup

### Get latest backup:
    
    $ export AWS_PROFILE=randstad-prod
    $ aws s3 ls gitlab-backups-service | sort | tail -n 1 | cut -d " " -f 5 | sed 's/_gitlab_backup.tar//'
    > 1685656813_2023_06_01_15.4.6-ee

The timestamp to extract is: `1685656813_2023_06_01_15.4.6-ee`

### Restore

    $ kubie ctx sandbox

    $ kubectl exec -it $(kubectl get po -l app=toolbox -n gitlab -o=jsonpath='{.items[].metadata.name}') -n gitlab -it -- env BACKUP_BUCKET_NAME=gitlab-backups-service backup-utility --restore -t 1686607207_2023_06_12_15.4.6-ee


### Connect to toolbox to disable ci_pipeline_schedules

    $ gitlab-rails dbconsole
    $ UPDATE ci_pipeline_schedules SET active='f';


# Upgrading gitlab
Upgrading gitlab is a tricky because ArgoCD is plugged on it.
Even Gitlab itself is sync via ArgoCD and via itself.

So you need to:

## 1. Make a backup of gitlab
TODO
## 2. Push you code on another repository
TODO
