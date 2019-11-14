pipeline {
    agent any
    tools {
        maven 'mvn'
        jdk 'jdk'
    }
	environment {
        HARBOR  = credentials('harbor')
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
                             docker build -t harbor.run.haas-445.pez.pivotal.io/jenkins/spring-boot-service --build-arg JAR_FILE=gs-rest-service-0.1.0.jar
                             docker login https://harbor.run.haas-445.pez.pivotal.io/jenkins -u $HARBOR_USR -p $HARBOR_PWD
                             docker push harbor.run.haas-445.pez.pivotal.io/jenkins/spring-boot-service
                          '''
                    }

            }
    }
}
