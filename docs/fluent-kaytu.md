# Installing Fluent(Fluentbit) in Kaytu to send logs to Cloudwatch

Assuming we have Kaytu configured on a kubernetes cluster, we need the following components:

- An IAM account with the following permissions to write to cloudwatch logs.

- Credentials for the above account as a Kubernetes secret, in the cluster.

## Configuring IAM account

1. Create user with permissions to `create log groups`, `create log streams` and `write logs`. The following terraform templates can be used.

    ``` 
    # iamuser.tf
    resource "aws_iam_user" "cloudwatch_user" {
        name = "cloudwatch-user"
    }
    ```
    ```
    # iampolicy.tf
    resource "aws_iam_policy" "cloudwatch_policy" {
        name        = "CloudwatchLogs"
        description = "Policy to allow writing to Cloudwatch Logs"

        policy = jsonencode({
            Version = "2012-10-17"
            Statement = [
            {
                Effect = "Allow"
                Action = [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
                ]
                Resource = "*"
            }
            ]
        })

    }
    ```

    ```
    # iam_policy_attachement.tf
    resource "aws_iam_user_policy_attachment" "user_policy_attachment" {
    user       = aws_iam_user.cloudwatch_user.name
    policy_arn = aws_iam_policy.cloudwatch_policy.arn
    }
    ```

3. Create credentials for the above created user. Recommend using the AWS cli.

    ```
    aws iam create-access-key --user-name cloudwatch-user 
    ```


## Creating a Kubernetes secret for the credentials

`kubectl` can be used for handling the secret creation. 

1. Create a .env file:

    ```
    AWS_ACCESS_KEY_ID=<<access-key>>
    AWS_SECRET_ACCESS_KEY=<<secret-access-key>>
    ```

2. Verify current context in .kubeconfig.

    ```
    kubectl config current-context
    ```

3. Since the fluent operator is configured to reside in the `kaytu-octopus` namespace, add the credential secret in `kaytu-octopus` namespace.


    ```
    kubectl create secret generic cloudwatch-user -n kaytu-octopus --from-env-file=<path/to/.env>
    ```


## Result

Cloudwatch logs should have a log group `kaytu/application` with streams from all the installed application on Kaytu's cluster.

