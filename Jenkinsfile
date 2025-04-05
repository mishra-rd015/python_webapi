pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal-python'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'pythonwebapijenkins8387963808'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mishra-rd015/python_webapi.git'
            }
        }

        stage('Setup Python') {
            steps {
                bat 'python -m venv venv'
                bat 'call venv\\Scripts\\activate && python -m pip install --upgrade pip'
                bat 'call venv\\Scripts\\activate && pip install -r requirements.txt'
            }
        }

        stage('Package Application') {
            steps {
                bat 'powershell -Command "Remove-Item -Recurse -Force publish -ErrorAction SilentlyContinue"'
                bat 'powershell -Command "New-Item -ItemType Directory -Path publish"'
                bat 'powershell -Command "Copy-Item -Path \\"app.py\\",\\"requirements.txt\\",\\"startup.txt\\" -Destination publish -Force"'
                bat 'powershell -Command "Compress-Archive -Path publish\\* -DestinationPath publish.zip -Force"'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: "${env.AZURE_CREDENTIALS_ID}")]) {
                    bat '''
                        az login --service-principal -u "%AZURE_CLIENT_ID%" -p "%AZURE_CLIENT_SECRET%" --tenant "%AZURE_TENANT_ID%"
                        az webapp config appsettings set --resource-group "%RESOURCE_GROUP%" --name "%APP_SERVICE_NAME%" --settings SCM_DO_BUILD_DURING_DEPLOYMENT=true PORT=8000
                        az webapp deploy --resource-group "%RESOURCE_GROUP%" --name "%APP_SERVICE_NAME%" --src-path publish.zip --type zip
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed. Check logs above.'
        }
    }
}
