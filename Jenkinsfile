pipeline {
    agent any

    environment {
        AZURE_WEBAPP_NAME = 'flaskwebapi-rd015'
        AZURE_RESOURCE_GROUP = 'flask-rg'
        AZURE_APP_PLAN = 'flaskappplan'
        AZURE_LOCATION = 'eastus'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/mishra-rd015/python_webapi.git', branch: 'main'
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
                bat '''
                    if exist publish (
                        powershell -Command "Remove-Item -Recurse -Force publish"
                    )
                    mkdir publish
                    xcopy /E /I /Y app publish\\app
                    copy app.py publish\\
                    copy requirements.txt publish\\
                    copy web.config publish\\
                    powershell Compress-Archive -Path publish\\* -DestinationPath publish.zip
                '''
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure-service-principal-python',
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID'
                )]) {
                    bat '''
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        az webapp deployment source config-zip --resource-group %AZURE_RESOURCE_GROUP% --name %AZURE_WEBAPP_NAME% --src publish.zip
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
