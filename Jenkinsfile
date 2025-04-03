pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'AzureclassAssignThu'
        APP_SERVICE_PLAN = 'YourAppServicePlan'
        APP_LOCATION = 'eastus' // Change if needed
        DOTNET_RUNTIME = 'DOTNET|8.0' // Update runtime version if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/Yash2433/AzureclassAssignThu.git'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet restore'
                bat 'dotnet build --configuration Release'
                bat 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Ensure Azure Web App Exists') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    script {
                        def checkApp = bat(script: "az webapp list --resource-group $RESOURCE_GROUP --query \"[].name\" --output tsv", returnStdout: true).trim()
                        if (!checkApp.contains(APP_SERVICE_NAME)) {
                            echo "Azure Web App does not exist, creating it..."
                            bat "az group create --name $RESOURCE_GROUP --location $APP_LOCATION"
                            bat "az appservice plan create --name $APP_SERVICE_PLAN --resource-group $RESOURCE_GROUP --sku F1"
                            bat "az webapp create --resource-group $RESOURCE_GROUP --plan $APP_SERVICE_PLAN --name $APP_SERVICE_NAME --runtime $DOTNET_RUNTIME"
                        } else {
                            echo "Azure Web App already exists, proceeding with deployment..."
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID"
                    bat "powershell Compress-Archive -Path ./publish/* -DestinationPath ./publish.zip -Force"
                    bat "az webapp deploy --resource-group $RESOURCE_GROUP --name $APP_SERVICE_NAME --src-path ./publish.zip --type zip"
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
