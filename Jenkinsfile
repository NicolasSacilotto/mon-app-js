pipeline {
    agent any
    
    parameters {
        choice(
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement de déploiement',
            name: 'ENVIRONMENT'
        )
        booleanParam(
            defaultValue: false,
            description: 'Ignorer les tests ?',
            name: 'SKIP_TESTS'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Construction de l\'application'
                // Ajoutez vos commandes de build ici
            }
        }

        stage('Test') {
            when {
                expression { !params.SKIP_TESTS }
            }
            steps {
                echo 'Exécution des tests'
                // Ajoutez vos commandes de test ici
            }
        }

        stage('Deploy') {
            steps {
                echo "Déploiement vers ${params.ENVIRONMENT}"
                script {
                    if (params.SKIP_TESTS) {
                        echo "Tests ignorés"
                    } else {
                        echo "Exécution des tests"
                    }
                }
                // Ajoutez vos commandes de déploiement ici
            }
        }
    }
    
    post {
        success {
            slackSend(
                channel: '#jenkins-notifications',
                color: 'good',
                attachments: [[
                    title: "Build ${env.BUILD_NUMBER} - ${env.JOB_NAME}",
                    titleLink: "${env.BUILD_URL}",
                    fields: [
                        [title: 'Branche', value: "${env.BRANCH_NAME}", short: true],
                        [title: 'Commit', value: "${env.GIT_COMMIT[0..7]}", short: true],
                        [title: 'Durée', value: "${currentBuild.durationString}", short: true]
                    ],
                    actions: [
                        [
                            type: "button",
                            text: "Voir dans Blue Ocean",
                            url: "${env.BUILD_URL}display/redirect"
                        ]
                    ]
                ]]
            )
        }
        
        failure {
            slackSend(
                channel: '#jenkins-notifications',
                color: 'danger',
                message: """
                Build échoué !
                Projet: ${env.JOB_NAME}
                Build: ${env.BUILD_NUMBER}
                Voir: ${env.BUILD_URL}
                """
            )
        }
    }
}