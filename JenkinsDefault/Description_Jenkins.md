# Jenkins on Kubernetes

## Runtime parameter mit values und freitext input feld
## pipeline templating
## build agent setup
## pipeline trigger
## hashicorp vault/azure keyvault integration
Jenkins provides a hashicorp vault plugin, as well as a azure key vault plugion to set environment variables from a HashiCorp/Azure Key Vault secret.
- configurable via jenkinsfile
```yaml
    node {
        // define the secrets and the env variables
        // engine version can be defined on secret, job, folder or global.
        // the default is engine version 2 unless otherwise specified globally.
        def secrets = [
            [path: 'secret/testing', engineVersion: 1, secretValues: [
                [envVar: 'testing', vaultKey: 'value_one'],
                [envVar: 'testing_again', vaultKey: 'value_two']]],
            [path: 'secret/another_test', engineVersion: 2, secretValues: [
                [vaultKey: 'another_test']]]
        ]

        // optional configuration, if you do not provide this the next higher configuration
        // (e.g. folder or global) will be used
        def configuration = [vaultUrl: 'http://my-very-other-vault-url.com',
                            vaultCredentialId: 'my-vault-cred-id',
                            engineVersion: 1]
        // inside this block your credentials will be available as env variables
        withVault([configuration: configuration, vaultSecrets: secrets]) {
            sh 'echo $testing'
            sh 'echo $testing_again'
            sh 'echo $another_test'
        }
    }
```
## deployment
- set up multiple pipelines via ui or groovy script
```yaml
pipeline {
    agent any

    stages {
        stage ('Deployment Stage') {
            steps {
                withMaven(maven : 'maven_3_5_0') {
                    sh 'mvn deploy'
                }
            }
        }
    }
}
```