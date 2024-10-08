

  eyJhbGciOiJSUzI1NiIsImtpZCI6ImhwcDBYeDJBemRwTkdCYVlkeTdIaVBvME5kNC11eGl4MjcwU1R3WDdEbHMifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im15c2VjcmV0bmFtZSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZjQ3MzgyYzItMGZlMy00NmU4LWE5YTktYzc0MDBjMDI3N2NiIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OndlYmFwcHM6amVua2lucyJ9.fxnQ2ocNiLe0L0xu4yyID6IUMT30Uobzr5fFSGhCShBSPpmyyVxU9RMW_kDhGxkP32HoXoPJeq0FLvUxk0aP7NhMi5_81ci5Zh3jdsdi9ywuw2Ou_5ib3mc7RgY28jHgQvq1j4eX1wDPSR_DVbNLHHQx5ER7UGrf45Y0rtOsMzWOm_TFCSDtm3jkEkh0YHCZoyCTA1EHL0OeaC4xltsytM1O9nNDO_8snPghYongT2hqBSsXBat51IQ8oWJMTQs87z0xq6G4ZQYAx5ooICK6Pa8okrHrKBIYkgL4piS77K8gPcUKBbgZXjG7E2T1N1wArJAPI7vo3NDMAPmEgqOz8A
  
  sonar-token- squ_9dca180f46e806bcff86116f4d199c9b91b6fb5f
  
  git-token- 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
  
  
  pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    } 
    environment {
        IMAGE_NAME = "adijaiswal/bankapp"
        TAG = "${params.DOCKER_TAG}"
        KUBE_NAMESPACE = 'webapps'
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git checkout') {
            steps {
              git branch: 'main', url: 'https://github.com/kishoretk12/Blue-Green-Deployment.git' 
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Maven test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('Trivy FS scan') {
            steps {
                sh "trivy fs --format -o fs.html ."
            }
        }
        stage('sonar-qube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Multitier -Dsonar.projectName=Multitier -Dsonar.java.binaries-target"
}
            }
        }
        stage('Quality-Gate check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false
}
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Publish Artifcat to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                 sh "mvn deploy -DskipTests=true"
}
            }
        }
        stage('Docker-bulid & tag Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                }
            }
        }
        
         stage('Trivy Image scan') {
            steps {
                sh "trivy fs --format -o fs.html ${IMAGE_NAME}:${TAG}"
            }
        }
        stage('Docker Image Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push ${IMAGE_NAME}:${TAG}"
                    }
                }
                stage('Deploy MySQL Deployment and Service') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C582DE09421DCD7DD20C18A7B63125D8.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f mysql-ds.yml -n ${KUBE_NAMESPACE}"  // Ensure you have the MySQL deployment YAML ready
                    }
                }
            }
        }
        stage('Deploy SVC-APP') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C582DE09421DCD7DD20C18A7B63125D8.gr7.us-east-1.eks.amazonaws.com') {
                        sh """ if ! kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}; then
                                kubectl apply -f bankapp-service.yml -n ${KUBE_NAMESPACE}
                              fi
                        """
                   }
                }
                 stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentFile = ""
                    if (params.DEPLOY_ENV == 'blue') {
                        deploymentFile = 'app-deployment-blue.yml'
                    } else {
                        deploymentFile = 'app-deployment-green.yml'
                    }

                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C582DE09421DCD7DD20C18A7B63125D8.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f ${deploymentFile} -n ${KUBE_NAMESPACE}"
                    }
                }
            }
        }
        stage('Switch Traffic Between Blue & Green Environment') {
            when {
                expression { return params.SWITCH_TRAFFIC }
            }
            steps {
                script {
                    def newEnv = params.DEPLOY_ENV

                    // Always switch traffic based on DEPLOY_ENV
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C582DE09421DCD7DD20C18A7B63125D8.gr7.us-east-1.eks.amazonaws.com') {
                        sh '''
                            kubectl patch service bankapp-service -p "{\\"spec\\": {\\"selector\\": {\\"app\\": \\"bankapp\\", \\"version\\": \\"''' + newEnv + '''\\"}}}" -n ${KUBE_NAMESPACE}
                        '''
                    }
                    echo "Traffic has been switched to the ${newEnv} environment."
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    def verifyEnv = params.DEPLOY_ENV
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://C582DE09421DCD7DD20C18A7B63125D8.gr7.us-east-1.eks.amazonaws.com') {
                        sh """
                        kubectl get pods -l version=${verifyEnv} -n ${KUBE_NAMESPACE}
                        kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
}

