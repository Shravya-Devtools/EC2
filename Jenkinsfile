pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
    }

    parameters {
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Terraform action to perform')
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/Shravya-Devtools/EC2.git', branch: 'master'
            }
        }

        stage('Setup AWS Credentials') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    echo 'AWS credentials injected'
                }
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }

        stage('Terraform Apply/Destroy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    script {
                        if (params.action == 'apply') {
                            input message: 'Apply the Terraform plan?', parameters: [text(name: 'Plan Preview', defaultValue: readFile('tfplan.txt'))]
                            sh 'terraform apply -auto-approve tfplan'
                        } else if (params.action == 'destroy') {
                            input message: 'Destroy resources?'
                            sh 'terraform destroy -auto-approve'
                        }
                    }
                }
            }
        }
    }
}
