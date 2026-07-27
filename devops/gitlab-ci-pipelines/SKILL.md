---
name: gitlab-ci-pipelines
description: 
category: devops
tags: [gitlab-ci-pipelines]
---

## When to Use
Build CI/CD pipelines with GitLab CI/CD for testing, building, deploying, and releasing. Covers `.gitlab-ci.yml`, stages, jobs, runners, caching, artifacts, environments, and auto-devops.

## Core Concepts
- **Stages**: Sequential phases (build → test → deploy)
- **Jobs**: Units of work within stages (can be parallel)
- **Runners**: Executors — shell, Docker, Kubernetes, or custom
- **Cache/Artifacts**: Cache for dependencies between jobs, artifacts for passing between stages
- **Variables**: CI/CD variables at project/group/instance level
- **Environments**: Deploy targets with stop/rollback support

## Workflow
1. Define stages in `.gitlab-ci.yml`
2. Write jobs for lint, test, build, deploy
3. Configure Docker-in-Docker or Kaniko for image builds
4. Set up environment protection rules
5. Use merge request pipelines for branch deploys
6. Tag releases for production deployments

## Key Patterns
```yaml
# .gitlab-ci.yml — Full pipeline
stages:
  - test
  - build
  - deploy-staging
  - deploy-production

variables:
  DOCKER_TLS_CERTDIR: "/certs"
  REGISTRY: registry.example.com

# Cache npm between jobs
.cache_template: &cache_template
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/

test:
  stage: test
  <<: *cache_template
  image: node:20-alpine
  script:
    - npm ci --cache .npm
    - npm run lint
    - npm test
    - npm run build
  artifacts:
    reports:
      junit: test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura.xml
    paths:
      - dist/
    expire_in: 1 hour

build-image:
  stage: build
  image: docker:24-dind
  services:
    - docker:24-dind
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build --cache-from $CI_REGISTRY_IMAGE:latest -t $IMAGE_TAG -t $CI_REGISTRY_IMAGE:latest .
    - docker push $IMAGE_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

deploy-staging:
  stage: deploy-staging
  image: bitnami/kubectl
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - kubectl set image deployment/api api=$IMAGE_TAG -n staging
    - kubectl rollout status deployment/api -n staging --timeout=300s
  only:
    - main

deploy-production:
  stage: deploy-production
  image: bitnami/kubectl
  environment:
    name: production
    url: https://api.example.com
  script:
    - kubectl set image deployment/api api=$IMAGE_TAG -n production
    - kubectl rollout status deployment/api -n production --timeout=300s
  when: manual
  only:
    - /^v\d+\.\d+\.\d+$/
  allow_failure: false
```

```yaml
# Merge request pipeline with review apps
review:
  stage: deploy-staging
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: stop-review
  script:
    - helm upgrade --install review-$CI_COMMIT_REF_SLUG ./chart
      --set image.tag=$CI_COMMIT_SHA
      --set ingress.host=$CI_COMMIT_REF_SLUG.review.example.com
  only:
    - merge_requests

stop-review:
  stage: deploy-staging
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  script:
    - helm uninstall review-$CI_COMMIT_REF_SLUG
  when: manual
  only:
    - merge_requests
```

## Pitfalls
- **DinD vs Kaniko**: DinD requires privileged runners; Kaniko is rootless
- **Cache key**: Use file-based keys (`key: { files: [...] }`) not branch names
- **Artifacts expiration**: Set expire_in to avoid disk bloat
- **Protected variables**: Use "Protected" flag for production secrets
- **Runner tags**: Tag runners for specific job types (docker, k8s)
- **Pipeline efficiency**: Use `rules:` instead of `only/except` (more flexible)

## Verification
```bash
# Validate YAML locally
gitlab-ci-lint  # or use GitLab CI/CD Lint UI

# Check pipeline status
glab ci list --limit 10
glab ci view <pipeline-id>

# Test pipeline locally
gitlab-runner exec docker test

# Verify runner registration
gitlab-runner list
```