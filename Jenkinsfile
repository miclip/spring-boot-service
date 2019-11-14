pipeline {
    agent any
    tools {
        maven 'mvn'
        jdk 'jdk'
    }
	environment {
        CICD_USER  = credentials('cicd_user')
        HARBOR  = credentials('harbor')
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "HARBOR_USR = ${HARBOR_USR}"
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
					withCredentials([certificate(aliasVariable: '', credentialsId: 'cicd_user', keystoreVariable: 'CICD_USER_CERT', passwordVariable: 'CICD_USER_CERT_PASSWORD')]) {
                       // some block
                     }
                }
            }
        }
    }
}
