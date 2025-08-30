pipeline{
    agent{
        label "AGENT-1"
    }
    options{        
        // Timeout counter starts before agent is allocated
        timeout(time: 30, unit: "MINUTES")
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        def appVersion = "" //variable declaration
        //nexusUrl = 'nexus.daws80s.online:8081'
        region = "us-east-1"
        account_id = "196900110977"
    }

    stages{
        stage("read json"){
            steps{
                script{ 
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version is $appVersion"
                }
            }
        }
        stage("install dependencies"){
            steps{
                sh """
                    npm install
                    ls -ltr
                    echo "application version is $appVersion" 
                """
           }
        }
        stage("Build"){
            steps{
                sh """
                  zip -q -r frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                  ls -ltr
                """
            }
            
        }
        stage("Deploy"){
            steps{
            sh """
                aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com
                docker build -t ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion} .
                docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/expense-frontend:${appVersion}
            """
            }
        }
        stage("helm install"){
            steps{
            sh """
                aws eks update-kubeconfig --region us-east-1 --name expense-dev
                cd helm
                sed -i 's/Image_Version/${appVersion}/g' values.yaml
                helm upgrade frontend .
            """
            }
        }
        // stage('Nexus Artifact Upload'){
        //     steps{
        //         script{
        //             nexusArtifactUploader(
        //                 nexusVersion: 'nexus3',
        //                 protocol: 'http',
        //                 nexusUrl: "${nexusUrl}",
        //                 groupId: 'com.expense',
        //                 version: "${appVersion}",
        //                 repository: "frontend",
        //                 credentialsId: 'nexus-auth',
        //                 artifacts: [
        //                     [artifactId: "frontend" ,
        //                     classifier: '',
        //                     file: "frontend-" + "$appVersion" + '.zip',
        //                     type: 'zip']
        //                 ]
        //             )
        //         }
        //     }
        // }
        // stage('Deploy'){
        //     steps{
        //         script{
        //             def params = [
        //                 string(name: 'appVersion', value: "${appVersion}")
        //             ]
        //             build job: 'frontend-deploy', parameters: params, wait: false
        //         }
        //     }
        // }

    }
    post{
        always{
            echo "this will run always"
            deleteDir()
        }
        success{
            echo "this will run only if build success"
        }
        failure{
            echo "this will run only if build fails"
        }
    }
}


