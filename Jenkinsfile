pipeline {
    agent { label 'docker' }
    options {
        disableConcurrentBuilds()
        overrideIndexTriggers(false)
    }

    triggers {
        cron('H/15 * * * *')
    }

    stages {
        stage('Update DNS') {
            steps {
                withCredentials([
                    file(credentialsId: 'ddns-sync.env', variable: 'ENV_FILE')
                ]) {
                    sh '''
                    set -e

                    source "$ENV_FILE"

                    CURRENT_IP=$(curl -s -m 10 https://cloudflare.com/cdn-cgi/trace | grep -E '^ip=' | cut -d= -f2)

                    if [[ ! $CURRENT_IP =~ ^[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+$ ]]; then
                        echo "$(date): Error: Failed to fetch a valid public IP. Got: $CURRENT_IP"
                        exit 1
                    fi

                    RESOLVED_IP=$(dig +short @1.1.1.1 $DOMAIN)

                    if [ "$CURRENT_IP" == "$RESOLVED_IP" ]; then
                        echo "$(date): IP matches ($CURRENT_IP). No update needed."
                        exit 0
                    fi

                    echo "$(date): IP changed from $RESOLVED_IP to $CURRENT_IP. Updating Cloudflare..."

                    RESPONSE=$(curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
                         -H "Authorization: Bearer $API_TOKEN" \
                         -H "Content-Type: application/json" \
                         --data "{\\"type\\":\\"A\\",\\"name\\":\\"$DOMAIN\\",\\"content\\":\\"$CURRENT_IP\\",\\"ttl\\":1,\\"proxied\\":false}")

                    if echo "$RESPONSE" | grep -q '"success":true'; then
                        echo "$(date): Update successful!"
                    else
                        echo "$(date): Update failed. API Response:"
                        echo "$RESPONSE"
                        exit 1
                    fi
                    '''
                }
            }
        }
    }

    post {
        failure {
            mail to: "${env.ALERT_EMAIL}",
                 subject: "🚨 DNS Update Failure - Build #${env.BUILD_NUMBER}",
                 body: "DNS update failed!\n\nCheck Jenkins logs here: ${env.BUILD_URL}"
        }
    }
}
