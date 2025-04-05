pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'linreactapptanishq'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/TanishqJecrc/ReactAppJenkins.git'
            }
        }

    stage('Setup Node Environment') {
            steps {
                dir('demo') {
                    bat 'node -v'
                    bat 'npm -v'
                    bat 'npm install -g npm@latest'
                }
            }
        }

        
        stage('Clean & Install Dependencies') {
            steps {
                dir('demo') {
                    bat 'del package-lock.json' // Clean previous installs
                    bat 'npm install || echo "Retrying npm install..." && npm install' // Retry if fails
                    bat 'npm audit fix --force' // Fix vulnerabilities
                    
                }
            }
        }
        
        stage('ReactBuild') {
            steps {
                dir('demo') {
                    bat 'npm run build' // Build the React app
                    bat 'powershell Compress-Archive -Path ./build/* -DestinationPath build.zip' // Archive build output
                }
                
            }
        }
         stage('Terraform Init') {
                steps {
                    dir('Terraform') {
                        bat 'terraform init'
                    }
                }
          }
          stage('Terraform Plan & Apply') {
            steps {
                dir('Terraform') {
                    bat 'terraform plan -out=tfplan'
                    bat 'terraform apply -auto-approve tfplan'
                }
            }
        }

        


        stage('Deploy') {
            steps {
               dir('WebAPI') {
                    withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    
                    bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./build.zip --type zip"
                }
                }
                   
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
