pipeline {
    agent {
        label 'laravel'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                checkout scm

                echo 'Installing system dependencies...'
                sh 'php -m | grep -i sqlite || echo "SQLite PHP extension not found in this build"'

                echo 'Copy env file...'
                sh 'cp .env.example .env'

                echo 'Configuring database...'
                sh 'sed -i "s|DB_HOST=.*|DB_HOST=localhost|" .env'
                sh 'sed -i "s|DB_PORT=.*|DB_PORT=3306|" .env'
                sh 'sed -i "s|DB_DATABASE=.*|DB_DATABASE=laravel|" .env'
                sh 'sed -i "s|DB_USERNAME=.*|DB_USERNAME=root|" .env'
                sh 'sed -i "s|DB_PASSWORD=.*|DB_PASSWORD=|" .env'

                echo 'Installing dependencies...'
                sh 'composer install'
                sh 'npm install'
                sh 'npm run build'

                echo 'Verifying Vite build...'
                sh 'test -f public/build/manifest.json || (echo "Vite manifest not found"; exit 1)'

                echo 'Generating application key...'
                sh 'php artisan key:generate'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                echo 'Checking PHP configuration...'
                sh '''
                    php -m | grep -qiE 'sqlite3|pdo_sqlite' || { echo "SQLite extension missing"; exit 1; }
                    php -r "if (!in_array('sqlite', PDO::getAvailableDrivers(), true)) { fwrite(STDERR, 'PDO sqlite driver missing'.PHP_EOL); exit(1); } echo 'PDO drivers: '.implode(', ', PDO::getAvailableDrivers()).PHP_EOL;"
                    php artisan test --parallel
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to 178.128.93.188/kimheng...'
                sh '''
                    DEPLOY_HOST="178.128.93.188"
                    TARGET_USER="${DEPLOY_USER:-root}"
                    TARGET_PASS="${DEPLOY_PASSWORD:-I4@2026GIC}"
                    INVENTORY_FILE="$(mktemp)"
                    echo "[web]" > "$INVENTORY_FILE"

                    HOST_LINE="$DEPLOY_HOST ansible_user=$TARGET_USER ansible_ssh_common_args='-o StrictHostKeyChecking=accept-new'"

                    if [ -n "$DEPLOY_KEY_PATH" ]; then
                        HOST_LINE="$HOST_LINE ansible_ssh_private_key_file=$DEPLOY_KEY_PATH"
                    elif [ -f "$HOME/.ssh/id_rsa" ]; then
                        HOST_LINE="$HOST_LINE ansible_ssh_private_key_file=$HOME/.ssh/id_rsa"
                    elif [ -n "$TARGET_PASS" ]; then
                        HOST_LINE="$HOST_LINE ansible_password=$TARGET_PASS ansible_ssh_pass=$TARGET_PASS"
                    else
                        echo "No deployment credentials found. Set DEPLOY_KEY_PATH or DEPLOY_PASSWORD in Jenkins."
                        exit 1
                    fi

                    echo "$HOST_LINE" >> "$INVENTORY_FILE"
                    echo "Using deploy user: $TARGET_USER"

                    ansible-playbook -i "$INVENTORY_FILE" deploy.yml -e "workspace=$WORKSPACE"
                '''
            }
        }
    }

    post {
        failure {
            script {
                def subject = "Jenkins FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                def body = """
Build failed.
Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}
""".stripIndent()

                try {
                    emailext(
                        to: "${env.NOTIFY_EMAILS ?: 'boypoor302@gmail.com'}",
                        subject: subject,
                        body: body
                    )
                    echo 'Failure email notification attempted.'
                } catch (err) {
                    echo "Email notification failed: ${err}"
                }

                sh '''
                    if [ -n "$TELEGRAM_BOT_TOKEN" ] && [ -n "$TELEGRAM_CHAT_ID" ]; then
                      TEXT="Jenkins FAILED: ${JOB_NAME} #${BUILD_NUMBER}%0A${BUILD_URL}"
                      curl -sS -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
                        -d chat_id="${TELEGRAM_CHAT_ID}" \
                        -d text="$TEXT" >/dev/null
                      echo "Telegram notification sent."
                    else
                      echo "Telegram notification skipped (set TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID)."
                    fi
                '''
            }
        }
    }
}