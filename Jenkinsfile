pipeline {
    agent any

    parameters {
        booleanParam(name: 'DESTROY', defaultValue: false, description: 'Check to destroy the infrastructure')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'us-east-1' // Change to your region
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Plan') {
            when {
                expression { !params.DESTROY }
            }
            steps {
                sh 'terraform plan -out=plan.tfout'
            }
        }

        stage('Approval') {
            when {
                expression { !params.DESTROY }
            }
            steps {
                input message: 'Apply Terraform?', ok: 'Apply'
            }
        }

        stage('Terraform Apply') {
            when {
                expression { !params.DESTROY }
            }
            steps {
                sh 'terraform apply -auto-approve plan.tfout'
            }
        }

        stage('Terraform Destroy') {
            when {
                expression { params.DESTROY }
            }
            steps {
                sh 'terraform destroy -auto-approve'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
