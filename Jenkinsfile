import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
    def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
    for (p in pubProfiles) {
        if (p['publishMethod'] == 'FTP') {
            return [url: p.publishUrl, username: p.userName, password: p.userPWD]
        }
    }
    error("No FTP publish profile found.") // Added error handling
}

node {
    withEnv(['AZURE_SUBSCRIPTION_ID=1139ba69-0311-419a-9f20-d3c173f69bd6',
              'AZURE_TENANT_ID=bbbe982c-a392-4062-8d84-0bba62bc7911']) {
        stage('init') {
            checkout scm // Checking out source code
        }
  
        stage('build') {
            sh 'mvn clean package' // Building the project
        }
  
        stage('deploy') {
            def resourceGroup = 'jenkins-get-started-rg'
            def webAppName = 'ShuaihangZhang'
            
            // Login to Azure
            withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
                sh '''
                    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
                    az account set -s $AZURE_SUBSCRIPTION_ID
                '''
            }
            
            // Get publish settings
            def pubProfilesJson = sh(script: "az webapp deployment list-publishing-profiles -g ${resourceGroup} -n ${webAppName} --output json", returnStdout: true).trim()
            def ftpProfile = getFtpPublishProfile(pubProfilesJson)
            
            // Upload package
            sh "az webapp deploy --resource-group ${resourceGroup} --name ${webAppName} --src-path target/calculator-1.0.war --type war"
            sh 'az logout' // Log out from Azure
        }
    }
}
