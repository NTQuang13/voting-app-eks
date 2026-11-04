# EKS application manifests

This directory is a Kustomize root for the voting application. It assumes the AWS Load Balancer Controller and a default StorageClass backed by persistent storage are installed in the EKS cluster.

Before Argo CD or `kubectl apply -k k8s-specifications` syncs it:

1. Build and push `vote`, `result`, and `worker` images to your ECR repositories. Use `docker build --target final -t <vote-image> vote` for `vote`; Compose uses its `dev` stage locally. Replace the example account, region, and `replace-me` tags in the three deployment files with immutable image references (prefer digests).
2. Replace `vote.example.com` and `result.example.com` in `ingress.yaml` with your DNS names. Configure DNS to point to the ALB created by the Ingress. Add HTTPS and a certificate before exposing the application publicly.
3. Provision a `db-credentials` Secret with a `password` key in the `voting` namespace, preferably through your secret management integration. For a temporary manual bootstrap: `kubectl create namespace voting` followed by `kubectl -n voting create secret generic db-credentials --from-literal=password='<password>'`.
4. Ensure the default StorageClass provisions volumes before starting PostgreSQL and Redis. The PVCs retain data across Pod restarts, but backups and recovery still need separate configuration.

The manifests use in-cluster PostgreSQL and Redis for a first EKS deployment. For production, use managed RDS and ElastiCache, update `DB_HOST` and `REDIS_HOST`, and remove their in-cluster resources. The applications also accept `DB_USER`, `DB_NAME`, and `DB_PASSWORD` through environment variables. Local Docker Compose retains its existing development defaults.
