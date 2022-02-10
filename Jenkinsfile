pipeline {
    agent {                                                             
        label 'SFBLD'
    }
    environment {
        SF_USERNAME_DEV = 'daniel-8299053017-25@industryapps.com'
        PRIVATE_KEY = credentials('SERVER_KEY_ID')
        CLIENT_ID_DEV = credentials('SF_CLIENT_ID_VLOCITY')
        SF_LOGIN_QA = 'https://login.salesforce.com'
        SF_DEV_PSW = credentials('SF_LOGIN_VLOCITY_PSW')
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '3'))
        skipDefaultCheckout()
        timeout(time: 120, unit: 'MINUTES')
    }
    stages {                                                           
        stage('Print Environment') {
            steps {
              
                 echo "Checking environment"
                 sh """
                 whoami
                 pwd
                 uname -a
                 ls
                 sfdx version
                 """
            }
        }
        stage('deploy to target') {
           
            steps {
                sh 'sfdx auth:jwt:grant -u daniel-8299053017-25@industryapps.com -f ${PRIVATE_KEY} -i ${CLIENT_ID_DEV} -r https://login.salesforce.com -a targetOrg'
                sh 'sfdx force:org:display -u ${SF_USERNAME_DEV}'
                echo "Checking out repository..."
                checkout scm
                sh 'tree'
                echo "Deploying ..."
                script {
                    echo "Creating delta packages"
                    //Create delta SF Folder
                    sh 'sfdx sgd:source:delta --to HEAD --from HEAD^ --output .'
                    sh 'ls -la'
                    echo "Deploying delta"
                    sh 'sfdx force:source:deploy -x package/package.xml '
                    echo "Deploying delta"
                    sh 'sfdx force:mdapi:deploy -d destructiveChanges  --ignorewarnings'
                   
                    
                        sh 'echo "###DONE!!"'
                   
                }
                sh 'sfdx force:org:display -u ${SF_USERNAME_DEV}'
                sh 'echo "sf.username=${SF_USERNAME_DEV}\n" > build.properties'
                sh 'echo "sf.password=${SF_DEV_PSW}\n" >> build.properties'
                sh 'echo "sf.loginUrl=https://login.salesforce.com\n" >> build.properties'
                sh 'echo "ignoreLWCActivationCards=true\n" >> build.properties'
                sh 'cat build.properties'
                sh 'vlocity -propertyfile build.properties -job ./Deploy_Delta.yaml packDeploy --verbose true --simpleLogging true'
            }
        }

       
    }
    post {
        always {
            echo 'cleaning up the workspace after build.....'
            deleteDir()
            sh 'echo "###DONE!!"'
            
        }
      
    }
}
