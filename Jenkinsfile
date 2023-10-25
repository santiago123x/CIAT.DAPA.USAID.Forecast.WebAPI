// Define an empty map for storing remote SSH connection parameters
def remote = [:]

pipeline {

    agent any

    environment {
        user = credentials('aclimate_user')
        host = credentials('aclimate_host')
        name = credentials('aclimate_name')
        ssh_key = credentials('aclimate_devops')
    }

    stages {
        stage('Ssh to connect Melisa server') {
            steps {
                script {
                    // Set up remote SSH connection parameters
                    remote.allowAnyHosts = true
                    remote.identityFile = ssh_key
                    remote.user = user
                    remote.name = name
                    remote.host = host
                    
                }
            }
        }
        stage('Deploy Aclimate WebApi') {
            steps {
                script {
                    sshCommand remote: remote, command: """
                        cd /webapps/
                        sudo /bin/systemctl stop aclimate-api
                        cp -r aclimate_webapi/* aclimate_webapi_bk/
                        rm -r aclimate_webapi/*
                        rm -fr releaseWebAPI.zip
                        curl -LOk https://github.com/santiago123x/CIAT.DAPA.USAID.Forecast.WebAPI/releases/latest/download/releaseWebAPI.zip
                        unzip -o releaseWebAPI.zip
                        rm -fr releaseWebAPI.zip
                        cp -r publish/WebAPI/* aclimate_webapi/
                        rm -r aclimate_webapi/appsettings.json
                        cp -r aclimate_webapi_bk/appsettings.json aclimate_webapi
                        cp -r aclimate_webapi_bk/Log/ aclimate_webapi
                        cp -r aclimate_webapi_bk/api.log aclimate_webapi
                        rm -fr publish
                        sudo /bin/systemctl start aclimate-api
                    """
                }
            }
        }
    }
    
    post {
        failure {
            script {
                    sshCommand remote: remote, command: """
                        cd /webapps/
                        sudo /bin/systemctl stop aclimate-api
                        rm -fr aclimate_webapi/*
                        cp -r aclimate_webapi_bk/* aclimate_webapi/
                        sudo /bin/systemctl start aclimate-api
                    """
                }
            }
        }

        success {
            script {
                echo 'everything went very well!!'
            }
        }
    }
 
}