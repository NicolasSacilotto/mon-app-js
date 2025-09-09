pipeline {
    agent any
    
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
            steps {
                echo 'Exécution des tests'
                // Ajoutez vos commandes de test ici
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo 'Déploiement en production'
                // Ajoutez vos commandes de déploiement ici
            }
        }
    }
    
    post {
        success {
            slackSend(
                channel: '#jenkins-notifications',
                color: 'good',
                message: """
                Build réussi !
                Projet: ${env.JOB_NAME}
                Build: ${env.BUILD_NUMBER}
                Branche: ${env.BRANCH_NAME}
                """
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