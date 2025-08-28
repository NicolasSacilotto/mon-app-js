pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18'
        APP_NAME = 'mon-app-js'
        DEPLOY_DIR = '/var/www/html/mon-app'
        DISCORD_WEBHOOK = 'https://discord.com/api/webhooks/1410545158226841653/3qNd98Usims2t5s6MO0dmE5EAX0S7whevJrgWWjhM-BOi2j-vUbePHuh75bGgoHjAgtZ'
    }

    stages {
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
        
        stage('Code Quality Check') {
            steps {
                echo 'Vérification de la qualité du code...'
                sh '''
                    echo "Vérification de la syntaxe JavaScript..."
                    find src -name "*.js" -exec node -c {} \\;
                    echo "Vérification terminée"
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
                sh '''
                    echo "Vérification des dépendances..."
                    npm audit --audit-level=high
                '''
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Déploiement vers l\'environnement de staging...'
                sh '''
                    echo "Déploiement staging simulé"
                    mkdir -p staging
                    cp -r dist/* staging/
                '''
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Déploiement vers la production...'
                sh '''
                    echo "Sauvegarde de la version précédente..."
                    if [ -d "${DEPLOY_DIR}" ]; then
                        cp -r ${DEPLOY_DIR} ${DEPLOY_DIR}_backup_$(date +%Y%m%d_%H%M%S)
                    fi
                    
                    echo "Déploiement de la nouvelle version..."
                    mkdir -p ${DEPLOY_DIR}
                    cp -r dist/* ${DEPLOY_DIR}/
                    
                    echo "Vérification du déploiement..."
                    ls -la ${DEPLOY_DIR}
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                echo 'Vérification de santé de l\'application...'
                script {
                    try {
                        sh '''
                            echo "Test de connectivité..."
                            # Simulation d'un health check
                            echo "Application déployée avec succès"
                        '''
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Warning: Health check failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }

    def sendDiscordMessage(String status, String color, String message) {
        sh """
            curl -H "Content-Type: application/json" \
                -X POST \
                -d '{
                    "embeds": [{
                        "title": "${status}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        "description": "${message}",
                        "color": ${color},
                        "url": "${env.BUILD_URL}"
                    }]
                    }' \
                ${env.DISCORD_WEBHOOK}
        """
    }

    post {
        always {
            echo 'Nettoyage des ressources temporaires...'
            sh '''
                rm -rf node_modules/.cache
                rm -rf staging
            '''
        }
        success {
            echo 'Pipeline exécuté avec succès!'  
            sendDiscordMessage("Succès", "3066992", "Le déploiement de ${env.JOB_NAME} s'est terminé avec succès.\nBranch: ${env.BRANCH_NAME}")
        }
        failure {
            echo 'Le pipeline a échoué!'
            sendDiscordMessage("Échec", "15158332", "Le déploiement de ${env.JOB_NAME} a échoué.\nBranch: ${env.BRANCH_NAME}")
        }
        unstable {
            echo 'Build instable - des avertissements ont été détectés'
            sendDiscordMessage("Instable", "16776960", "Des avertissements ont été détectés dans ${env.JOB_NAME}.\nBranch: ${env.BRANCH_NAME}")
        }
    }
}


