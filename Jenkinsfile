// pipeline {
//     agent { node { label "jenkins_cicd_slave" } }   
//     parameters {
//       choice(name: 'aws_account',choices: ['891377107358', '4568366404742', '922266408974','576900672829'], description: 'aws account hosting image registry')
//       choice(name: 'Environment', choices: ['Dev', 'QA', 'UAT', 'Prod'], description: 'Target environment for deployment')
//       string(name: 'ecr_tag', defaultValue: '1.7.0', description: 'Assign the ECR tag version for the build')
//     }

//     tools {
//       maven "maven-3.9.9"
//     }

//     stages {
//     stage('1. Git Checkout') {
//       steps {
//         git branch: 'main', credentialsId: 'github_pat', url: 'https://github.com/Atoki23/AddressBookApp.git' 
//       }
//     }
//     stage('2. Build with Maven') { 
//       steps {
//         sh "mvn clean package"
//       }
//     }
//     stage('3. SonarQube Analysis') {
//           environment {
//                 scannerHome = tool 'SonarQube-Scanner-6.2.1'
//             }
//             steps {
//               withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
//                       sh """
//                       ${scannerHome}/bin/sonar-scanner  \
//                       -Dsonar.projectKey=addressbook \
//                       -Dsonar.projectName='addressbook' \
//                       -Dsonar.host.url=http://3.8.130.11:9000 \
//                       -Dsonar.token=${SONAR_TOKEN} \
//                       -Dsonar.sources=src/main/java/ \
//                       -Dsonar.java.binaries=target/classes \
//                      """
//                   }
//               }
//         }
//     stage('4. Docker Image Build') {
//       steps {
//           sh "aws ecr get-login-password --region eu-west-2 | sudo docker login --username AWS --password-stdin ${params.aws_account}.dkr.ecr.eu-west-2.amazonaws.com"
//           sh "sudo docker build -t addressbook ."
//           sh "sudo docker tag addressbook:latest ${params.aws_account}.dkr.ecr.eu-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
//           sh "sudo docker push ${params.aws_account}.dkr.ecr.eu-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
//       }
//     }

    //  stage('5. Clean Up Docker Images') {
    //         steps {
    //             script {
    //                 // Removing all Docker images from the Docker server
    //                 sh "sudo docker rmi -f $(docker images -q)"
    //             }
    //         }
    //     }

//     stage('6. Application Deployment in EKS') {
//       steps {
//         withkubeconfig([caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '']) {
//           sh "kubectl apply -k manifest"
//         }
//       }
//     }

//     stage('7. Monitoring Solution Deployment in EKS') {
//       steps {
//         withkubeconfig([caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '']) {
//           sh "kubectl apply -k monitoring"
//           sh "chmod +x -R script
//           sh("""script/install_helm.sh""") 
//           sh("""script/install_prometheus.sh""") 
//         }
//       }
//     }

// //     stage('8. Email Notification') {
// //       steps {
// //         mail bcc: 'oluwaseunatoki1@gmail.com', body: '''Build is Over. Check the application using the URL below:
// //          https://app.dominionsystem.org/addressbook-1.0
// //          Let me know if the changes look okay.
// //          Thanks,
// //          Atoktech Nigeria Limited,
// //          +44743874984''', 
// //          subject: 'Application was Successfully Deployed!!', to: 'oluwaseunatoki1@gmail.com'
// //       }
// //     }
// //   }
// // }

pipeline {
    agent { 
        node { label "jenkins_cicd_slave" } 
    }   

    parameters {
        choice(
            name: 'aws_account',
            choices: ['891377107358', '4568366404742', '922266408974', '576900672829'],
            description: 'AWS account hosting image registry'
        )
        choice(
            name: 'Environment',
            choices: ['Dev', 'QA', 'UAT', 'Prod'],
            description: 'Target environment for deployment'
        )
        string(
            name: 'ecr_tag',
            defaultValue: '1.7.0',
            description: 'Assign the ECR tag version for the build'
        )
    }

    tools {
        maven "maven-3.9.9"
    }

    environment {
        SCANNER_HOME = tool('SonarQube-Scanner-6.2.1')
        SONAR_HOST = 'http://13.40.88.214:9000' 
        REGISTRY_URL = "${params.aws_account}.dkr.ecr.eu-west-2.amazonaws.com"
    }
    
stages {
        stage('1. Git Checkout') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github_pat', 
                    url: 'https://github.com/Atoki23/AddressBookApp.git'
            }
        }

        stage('2. Build with Maven') { 
            steps {
                sh "mvn clean package"
            }
        }

        stage('3. SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=addressbook \
                        -Dsonar.projectName='addressbook' \
                        -Dsonar.host.url=${SONAR_HOST} \
                        -Dsonar.token=${SONAR_TOKEN} \
                        -Dsonar.sources=src/main/java/ \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('4. Docker Image Build and Push') {
            steps {
                script {
                    try {
                        sh """
                            aws ecr get-login-password --region eu-west-2 | sudo docker login --username AWS --password-stdin ${REGISTRY_URL}
                            sudo docker build -t addressbook .
                            sudo docker tag addressbook:latest ${REGISTRY_URL}/addressbook:${params.ecr_tag}
                            sudo docker push ${REGISTRY_URL}/addressbook:${params.ecr_tag}
                        """
                    } catch (Exception e) {
                        error "Docker build or push failed: ${e.message}"
                    }
                }
            }
        }

        stage('5. Clean Up Docker Images') {
            steps {
                sh "sudo docker rmi -f \$(sudo docker images -q)" //Without the backslash, Jenkins would attempt to resolve $() as a Groovy variable or syntax, which would cause error
            }
        }

        stage('6. Application Deployment in EKS') {
            steps {
                withKubeConfig([caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '']) {
                    sh "kubectl apply -k manifest"
                }
            }
        }

        stage('7. Monitoring Solution Deployment in EKS') {
            steps {
                withKubeConfig([caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '']) {
                        sh "kubectl apply -k monitoring"
                        
                        sh("""script/install_helm.sh""") 
                        sh("""script/install_prometheus.sh""")  
                }
            }
        }

        stage('8. Email Notification') {
            steps {
                mail bcc: 'oluwaseunatoki1@gmail.com', 
                     body: '''Build is completed. Check the application using the URL below:
                     https://app.dominionsystem.org/addressbook-1.0
                     Let me know if the changes look okay.
                     Thanks,
                     Atoktech Nigeria Limited,
                     +44743874984''', 
                     subject: 'Application Successfully Deployed!',
                     to: 'oluwaseunatoki1@gmail.com'
            }
        }
    }
}
