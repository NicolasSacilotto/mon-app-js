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


      stage('Generate Coverage Report') {
            steps {
                echo 'Génération du rapport de couverture Cobertura...'
                dir("${env.WORKSPACE}") {
                    sh '''
                        npm test -- --coverage
                        echo "Liste des fichiers coverage/"
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
        publishCoverage adapters: [
            coberturaAdapter('coverage/**/*.xml')
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
    post {
        success {
            echo 'Pipeline exécuté avec succès!'
            script {
                sh """
                    export LANG=C.UTF-8
                    export LC_ALL=C.UTF-8
                    cat > payload.json <<EOF
    {
    "embeds": [
        {
        "title": "Succes: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        "description": "Le deploy de ${env.JOB_NAME} fonctionne.\\nBranch: ${env.BRANCH_NAME}",
        "color": 3066993,
        "url": "${env.BUILD_URL}"
        }
    ]
    }
    EOF
                    curl -H 'Content-Type: application/json; charset=utf-8' -X POST -d @payload.json ${env.DISCORD_WEBHOOK}
                    rm payload.json
                """
            }
        }

        failure {
            echo 'Le pipeline est en echec!'
            script {
                sh """
                    export LANG=C.UTF-8
                    export LC_ALL=C.UTF-8
                    cat > payload.json <<EOF
    {
    "embeds": [
        {
        "title": "Échec: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        "description": "Le deploy de ${env.JOB_NAME} ne fonctionne pas.\\nBranch: ${env.BRANCH_NAME}",
        "color": 15158332,
        "url": "${env.BUILD_URL}"
        }
    ]
    }
    EOF
                    curl -H 'Content-Type: application/json; charset=utf-8' -X POST -d @payload.json ${env.DISCORD_WEBHOOK}
                    rm payload.json
                """
            }
        }

        unstable {
            echo 'Build instable - Avertissements'
            script {
                sh """
                    export LANG=C.UTF-8
                    export LC_ALL=C.UTF-8
                    cat > payload.json <<EOF
    {
    "embeds": [
        {
        "title": "Instable: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        "description": "Avertissements dans ${env.JOB_NAME}.\\nBranch: ${env.BRANCH_NAME}",
        "color": 16776960,
        "url": "${env.BUILD_URL}"
        }
    ]
    }
    EOF
                    curl -H 'Content-Type: application/json; charset=utf-8' -X POST -d @payload.json ${env.DISCORD_WEBHOOK}
                    rm payload.json
                """
            }
        }
    }

}