```
git clone https://github.com/ehime/poc-vault-keystore.git vault
cd vault
docker-compose up -d
export VAULT_ADDR=http://192.168.99.100:8200
```

Initializing a vault:
```
vault init
vault unseal <secret 1>
vault unseal <secret 2>
vault unseal <secret 3>
```

Authorizing using the root token:
```
vault auth <root token>
```

### Dynamic AWS Credentials

https://www.vaultproject.io/docs/secrets/aws/index.html

```
$ vault mount aws
Successfully mounted 'aws' at 'aws'!

$ vault write aws/config/root \
    access_key=<aws_access_key_id> \
    secret_key=<aws_secret_access_key> \
    region=us-east-1

# use http://awspolicygen.s3.amazonaws.com/policygen.html to generate policies
# here is an example one which provides full access to <bucket name>:
vault write aws/roles/s3 name=s3 policy=- <<EOF
{
  "Statement" : [
    {
      "Effect" : "Allow",
      "Action" : [
        "s3:ListAllMyBuckets"
      ],
      "Resource" : "arn:aws:s3:::*"
    }, {
      "Effect" : "Allow",
      "Action" : [
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource" : "arn:aws:s3:::<bucket name>"
    }, {
      "Effect" : "Allow",
      "Action" : [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource":"arn:aws:s3:::<bucket name>/*"
    }
  ]
}
EOF

$ vault read aws/creds/s3
Key             Value
lease_id        aws/creds/s3/7cb8df71-782f-3de1-79dd-251778e49f58
lease_duration  3600
access_key      AKIAIOMYUTSLGJOGLHTQ
secret_key      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
```

### Dynamic MySQL Usernames/Passwords

https://www.vaultproject.io/docs/secrets/mysql/index.html

```
$ vault mount mysql
Successfully mounted 'mysql' at 'mysql'!

$ vault write mysql/config/connection value="root:secret@tcp(mysql:3306)/"
Success! Data written to: mysql/config/connection

$ vault write mysql/config/lease lease=1h lease_max=24h
Success! Data written to: mysql/config/lease

$ vault write mysql/roles/readonly sql="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';"
Success! Data written to: mysql/roles/readonly

$ vault read mysql/creds/readonly
Key             Value
lease_id        mysql/creds/readonly/bd404e98-0f35-b378-269a-b7770ef01897
lease_duration  3600
password        132ae3ef-5a64-7499-351e-bfe59f3a2a21
username        root-aefa635a-18
```
