pipeline {
    agent any
    tools {
        maven 'mvn'
        jdk 'jdk'
    }
	environment {
        HARBOR  = credentials('harbor')
        IMAGE_NAME = "harbor.run.haas-445.pez.pivotal.io/jenkins/spring-boot-service"
        REPO_URL= "https://harbor.run.haas-445.pez.pivotal.io/jenkins"
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }

        stage ('Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true install'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml'
                }
            }
        }

        stage ('Docker Build') {
             steps {
                         sh '''
                             cp "${WORKSPACE}/target/gs-rest-service-0.1.0.jar" "${WORKSPACE}/"
                             docker build -t $IMAGE_NAME . --build-arg JAR_FILE=gs-rest-service-0.1.0.jar
                             docker login $REPO_URL -u "$HARBOR_USR" -p "$HARBOR_PSW"
                             docker push $IMAGE_NAME
                          '''
                    }

            }
        stage ('Kubernetes Deploy') {
             steps {
                    withCredentials([certificate(aliasVariable: '', credentialsId: 'cicd_user', keystoreVariable: 'CICD_USER_CERT', passwordVariable: 'CICD_USER_CERT_PASSWORD')]) {
                         sh '''
                              openssl pkcs12 -in $CICD_USER_CERT -out cicd.key -nocerts -nodes -passin pass:$CICD_USER_CERT_PASSWORD
                              openssl pkcs12 -in $CICD_USER_CERT -out cicd.crt -clcerts -nokeys -passin pass:$CICD_USER_CERT_PASSWORD
                              kubectl config set-credentials cicd --client-certificate=cicd.crt --client-key=cicd.key
                              kubectl config set-context cicd-context --cluster=dev --user=cicd
                              kubectl apply -f "${WORKSPACE}/manifest.yml" -n apps
                          '''
                      }
                   }
            }
    }
}
