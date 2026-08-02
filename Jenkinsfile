def imageName = 'ttrend'
def version = '2.0.2'

pipeline {
    agent {
        node {
            label "valaxy"
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.8.7/bin:$PATH"
    }

    stages {

        stage('build') {
            steps {
                echo "------------ build started ---------"

                configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {

                    sh '''
                    echo "===== Maven Settings ====="
                    cat $MAVEN_SETTINGS
                    echo "=========================="

                    mvn -s $MAVEN_SETTINGS help:effective-settings

                    mvn clean package -Dmaven.test.skip=true -s $MAVEN_SETTINGS
                    '''
                }

                echo "------------ build completed ---------"
            }
        }

        stage('Unit Test') {
            steps {
                echo '<--------------- Unit Testing started --------------->'
                sh 'mvn surefire-report:report'
                echo '<--------------- Unit Testing stopped --------------->'
            }
        }

        stage("Sonar Analysis") {
            environment {
                scannerHome = tool 'valaxy-sonarscanner'
            }

            steps {
                echo '<--------------- Sonar Analysis started --------------->'

                withSonarQubeEnv('valaxy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }

                echo '<--------------- Sonar Analysis stopped --------------->'
            }
        }

        stage("Quality Gate") {
            steps {
                script {

                    echo '<--------------- Sonar Gate Analysis Started --------------->'

                    timeout(time: 1, unit: 'HOURS') {

                        def qg = waitForQualityGate()

                        if (qg.status != 'OK') {
                            error "Pipeline failed due to quality gate failures: ${qg.status}"
                        }
                    }

                    echo '<--------------- Sonar Gate Analysis Ends --------------->'
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {

                    echo '<--------------- Nexus Jar Publish Started --------------->'

                    configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {

                        sh '''
                        echo "===== Maven Settings Used During Deploy ====="
                        cat $MAVEN_SETTINGS
                        echo "============================================"

                        mvn -s $MAVEN_SETTINGS help:effective-settings

                        echo "Sleeping for 120 seconds..."
                        sleep 120

                        mvn -X deploy -Dmaven.test.skip=true -s $MAVEN_SETTINGS
                        '''
                    }

                    echo '<--------------- Nexus Jar Publish Completed --------------->'
                }
            }
        }

        stage(" Docker Build ") {
            steps {
                script {

                    echo '<--------------- Docker Build Started --------------->'

                    app = docker.build("${imageName}:${version}")

                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage(" Docker Publish ") {
            steps {
                script {

                    echo '<--------------- Docker Publish Skipped --------------->'
                    echo 'JFrog has been removed. Configure Docker Hub or Nexus Docker Registry later.'
                }
            }
        }

        stage(" Deploy ") {
            steps {
                script {

                    echo '<--------------- Deploy Started --------------->'

                    sh 'helm install twittertrend-2.0.2 ttrend'

                    echo '<--------------- Deploy Ends --------------->'
                }
            }
        }
    }
}