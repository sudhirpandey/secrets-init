# secrets-init

`secrets-init` is a minimalistic init system designed to run as PID 1 inside container environments, similar to [dumb-init](https://github.com/Yelp/dumb-init), integrated with [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) and [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) services.

## Why you need an init system

Please [read Yelp *dumb-init* repo explanation](https://github.com/Yelp/dumb-init/blob/v1.2.0/README.md#why-you-need-an-init-system)

Summary:

- Proper signal forwarding
- Orphaned zombies reaping

## What `secrets-init` does

`secrets-init` runs as `PID 1`, acting like a simple init system. It launches a single process and then proxies all received signals to a session rooted at that child process.

`secrets-init` also passes almost all environment variables without modification, replacing _secret variables_ with values from secret management services.

### Integration with AWS Secrets Manager

User can put AWS secret ARN as environment variable value. The `secrets-manager` will resolve any environment value, using specified ARN, to referenced secret value.

```sh
# environment variable passed to `secrets-init`
MY_DB_PASSWORD=arn:aws:secretsmanager:$AWS_REGION:$AWS_ACCOUNT_ID:secret:mydbpassword-cdma3

# environment variable passed to child process, resolved by `secrets-init`
MY_DB_PASSWORD=very-secret-password
```

### Integration with AWS Systems Manager Parameter Store

It is possible to use AWS Systems Manager Parameter Store to store application parameters and secrets.

User can put AWS Parameter Store ARN as environment variable value. The `secrets-manager` will resolve any environment value, using specified ARN, to referenced parameter value.

```sh
# environment variable passed to `secrets-init`
MY_API_KEY=arn:aws:ssm:$AWS_REGION:$AWS_ACCOUNT_ID:parameter/api/key

# environment variable passed to child process, resolved by `secrets-init`
MY_API_KEY=key-123456789
```

### Requirement

In order to resolve AWS secrets from AWS Secret Manager and Parameter Store, `secrets-init` should run under IAM role that has permission to access desired secrets.

This can be achieved by assigning IAM Role to Kubernetes Pod or ECS Task. It's possible to assign IAM Role to EC2 instance, where container is running, but this option is less secure.

## Code Reference

Initial init system code was copied from [go-init](https://github.com/pablo-ruth/go-init) project.
