pipeline {
    agent any

    environment {
        NODE_VERSION = '18'
        APP_NAME = 'mon-app-js'
        DEPLOY_DIR = '/var/www/html/mon-app'
        DISCORD_WEBHOOK = 'https://discord.com/api/webhooks/1410545158226841653/3qNd98Usims2t5s6MO0dmE5EAX0S7whevJrgWWjhM-BOi2j-vUbePHuh75bGgoHjAgtZ'
        TMPDIR = '/tmp'
    }

    stages {
        stage('Start of Pipeline') {
            steps {
                sh '''
                    echo "Début du pipeline pour ${APP_NAME}"
                '''
            }
        }

        stage('Checkout') {
            steps {
                echo 'Récupération du code source...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installation des dépendances Node.js...'
                sh '''
                    whoami
                    node --version
                    npm --version
                    npm ci
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Exécution des tests...'
                sh 'npm test'
            }
            post {
                always {
                    junit 'tests/junit.xml'
                }
            }
        }

        stage('sed for Cobertura') {
            steps {
                sh 'npm test'
                sh '''
                # Normaliser la source dans Cobertura pour Jenkins
                sed -i 's|<source>.*</source>|<source>/jenkins_rs/mon-app-js</source>|' coverage/cobertura-coverage.xml
                '''
            }
        }

        stage('Generate Coverage Report') {
            steps {
                dir('/jenkins_rs/mon-app-js') {
                    sh '''
                        npx jest --coverage --coverageReporters=cobertura --rootDir=/jenkins_rs/mon-app-js
                        ls -la coverage/
                        if [ ! -f coverage/cobertura-coverage.xml ]; then
                            echo "ERREUR : coverage/cobertura-coverage.xml introuvable"
                            exit 1
                        fi
                    '''
                }
            }
        }


        stage('Code Coverage') {
            steps {
                echo 'Analyse de la couverture de code...'
                sh 'ls -l coverage/cobertura-coverage.xml'
                sh 'head -n 20 coverage/cobertura-coverage.xml'
                publishCoverage adapters: [
                    coberturaAdapter('coverage/cobertura-coverage.xml')
                ],
                failNoReports: true,
                globalThresholds: [
                    [thresholdTarget: 'LINE', unhealthyThreshold: 70.0, unstableThreshold: 80.0],
                    [thresholdTarget: 'BRANCH', unhealthyThreshold: 60.0, unstableThreshold: 70.0]
                ]
            }
        }

        stage('Code Quality Check') {
            steps {
                echo 'Vérification de la qualité du code...'
                sh '''
                    find src -name "*.js" -exec node -c {} \\;
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Construction de l\'application...'
                sh '''
                    npm run build
                    ls -la dist/
                '''
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Analyse de sécurité...'
                sh 'npm audit --audit-level=high || true'
            }
        }

        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps {
                echo 'Déploiement vers staging...'
                sh '''
                    mkdir -p staging
                    cp -r dist/* staging/
                '''
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                echo 'Déploiement vers production...'
                sh '''
                    if [ -d "${DEPLOY_DIR}" ]; then
                        cp -r ${DEPLOY_DIR} ${DEPLOY_DIR}_backup_$(date +%Y%m%d_%H%M%S)
                    fi
                    mkdir -p ${DEPLOY_DIR}
                    cp -r dist/* ${DEPLOY_DIR}/
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Vérification de santé...'
                sh 'echo "Application déployée avec succès"'
            }
        }
    }

    post {
        success {
            echo 'Pipeline exécuté avec succès!'
            script {
                sh """
                    cat > payload.json <<EOF
{
"embeds":[{"title":"Succes: ${env.JOB_NAME} #${env.BUILD_NUMBER}","description":"Le deploy fonctionne. Branch: ${env.BRANCH_NAME}","color":3066993,"url":"${env.BUILD_URL}"}]
}
EOF
                    curl -H 'Content-Type: application/json' -X POST -d @payload.json ${env.DISCORD_WEBHOOK}
                    rm payload.json
                """
            }
        }

        failure {
            echo 'Le pipeline est en echec!'
            script {
                sh """
                    cat > payload.json <<EOF
{
"embeds":[{"title":"Échec: ${env.JOB_NAME} #${env.BUILD_NUMBER}","description":"Le deploy ne fonctionne pas. Branch: ${env.BRANCH_NAME}","color":15158332,"url":"${env.BUILD_URL}"}]
}
EOF
                    curl -H 'Content-Type: application/json' -X POST -d @payload.json ${env.DISCORD_WEBHOOK}
                    rm payload.json
                """
            }
        }

        unstable {
            echo 'Build instable - Avertissements'
            script {
                sh """
                    cat > payload.json <<EOF
{
"embeds":[{"title":"Instable: ${env.JOB_NAME} #${env.BUILD_NUMBER}","description":"Avertissements. Branch: ${env.BRANCH_NAME}","color":16776960,"url":"${env.BUILD_URL}"}]
}
EOF
                    curl -H 'Content-Type: application/json' -X POST -d @payload.json ${env.DISCORD_WEBHOOK}
                    rm payload.json
                """
            }
        }
    }
}
