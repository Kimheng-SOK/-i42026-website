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
                sh 'sudo apt-get update && sudo apt-get install -y php-sqlite3'

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
                sh 'php artisan test --parallel'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to 178.128.93.188/kimheng...'
                sh 'ansible-playbook -i inventory/hosts deploy.yml'
            }
        }
    }
}