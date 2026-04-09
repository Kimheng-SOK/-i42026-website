pipeline {
    agent {
        label 'laravel'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                checkout scm

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

                echo 'Generating application key...'
                sh 'php artisan key:generate'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'php artisan test'
            }
        }
        // stage('Deploy') {
        //     steps {
        //         echo 'Deploying...'
        //         sh 'ansible-playbook -i inventory/hosts deploy.yml'
        //     }
        // }
    }
}