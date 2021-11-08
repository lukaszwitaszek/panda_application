pipeline {
    agent {
        label 'node_1'
    }
    tools {
        maven 'auto_maven'
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    stages{
        stage("Clear runing apps"){
            steps {
                sh 'echo clear'
                sh 'docker rm -f pandaapp || true'
                //sh 'docker rm -f ${NAZWA_KONTENERA} || true'
                // zmienna do ustawienia w parametrized pipeline
            }
        }
        stage ('Get Code'){
            steps {
                sh 'echo get_code'
                //git credentialsid:'maven_at_artifactory', url: 'https://github.com/mwocka/panda_application.git'
                git branch: 'selenium_grid', url: 'https://github.com/mwocka/panda_application.git'
            }
        }
        stage('Build and Junit'){
            steps{
                sh 'echo build_and_Junit'
                sh 'mvn clean install'
            }
        }
        stage("Build Docker image"){
            steps {
                sh 'echo build_dockr_image'
                sh 'mvn package -Pdocker'
            }
        }
        stage("Test Selenium"){
            steps {
                script{
                    if (${selenium}) {
                        sh 'echo selenium'
                        sh "mvn test -Pselenium"
                    } else {
                        sh 'echo no_selenium'
                        sh 'true'
                    }
                }
            }
        }
        stage("Run Docker app"){
            steps {
                sh 'echo run_Docker_app'
                sh 'docker run -d -p 0.0.0.0:8080:8080 --name pandaapp ${IMAGE}:${VERSION}'
                
            }
        }
        stage("Deploy jar to artifactory"){
            steps {
                sh 'echo jar_to_arti'
                configFileProvider([configFile(fileId: '74c73f9b-9536-4ce4-a987-72c28a30c98e', variable: 'MAVEN_SETTINGS_XML')]) {
                    sh 'mvn -gs $MAVEN_SETTINGS_XML deploy -Dmaven.test.skip=true -e'
                }
            }
        }
    }
    post{
        always{
        sh "echo post_always"
        }
    }
}
