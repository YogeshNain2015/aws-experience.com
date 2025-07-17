# VPC Lattice

https://catalog.us-east-1.prod.workshops.aws/workshops/c5c86913-d1c9-4095-8b7f-cd3df11b5e1b/en-US  
```bash
curl -s web-svc-1-0aa8d5dd261fecb83.7d67968.vpc-lattice-svcs.us-east-1.on.aws/index.php \
            --aws-sigv4 "aws:amz:us-east-1:vpc-lattice-svcs" \
            --user "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \
            --header "x-amz-security-token:$AWS_SESSION_TOKEN" \
            --header "x-amz-date:$(date -u +%Y%m%dT%H%M%SZ)" \
            --header "x-amz-content-sha256:UNSIGNED-PAYLOAD"
```
## Get IMDSv2 token
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`

## First, get the role name 
AWS_ROLE_NAME=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/) 

## Then get credentials (with error checking)
CREDENTIALS=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" "http://169.254.169.254/latest/meta-data/iam/security-credentials/${AWS_ROLE_NAME}")

## Verify the JSON output 
echo "Credentials response: $CREDENTIALS"

## Extract individual credentials if JSON is valid 
if [ ! -z "$CREDENTIALS" ]; then
AWS_SESSION_TOKEN=$(echo $CREDENTIALS | jq -r '.Token')
AWS_ACCESS_KEY_ID=$(echo $CREDENTIALS | jq -r '.AccessKeyId')
AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIALS | jq -r '.SecretAccessKey')

echo "Session Token: ${AWS_SESSION_TOKEN:0:10}..."
echo "Access Key: ${AWS_ACCESS_KEY_ID:0:10}..."
echo "Secret Key: ${AWS_SECRET_ACCESS_KEY:0:10}..."
else
echo "Failed to retrieve credentials"
fi
