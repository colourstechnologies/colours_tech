pipeline{

    agent any

    stages{
        stage('Clone'){
            steps{
                git credentialsId: 'git-credentials', url: 'https://github.com/colourstechnologies/colours_tech.git'
            }
        }
        stage ('Build'){
            steps {
                script {
                def mavenHome = tool name: "Maven-3.9.4", type: "maven"
                def mavenCMD = "${mavenHome}/bin/mvn"
                sh "${mavenCMD} clean package"
                }
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube 8.9') {
                    def mavenHome = tool name: "Maven-3.9.4", type: "maven"
                    def mavenCMD = "${mavenHome}/bin/mvn"
                    sh "${mavenCMD} sonar:sonar"
                    }
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    artifacts: [[
                        artifactId: 'colours_tech',
                        classifier: '',
                        file: 'target/colours_tech.war',
                        type: 'war'
                    ]],
                    credentialsId: 'nexus-credentials',
                    groupId: 'in.colourstech',
                    nexusUrl: '3.108.223.52:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'colourstech-snapshot',
                    version: '1.0-SNAPSHOT'
                )
            }
        }
        stage ('Deploy') {
            steps {
                sshagent(['tomcat-ssh-agent']) {
                sh 'scp -o StrictHostKeyChecking=no target/colours_tech.war ec2-user@3.111.149.16:/tmp/'
                sh 'ssh -o StrictHostKeyChecking=no ec2-user@3.111.149.16 "sudo mv /tmp/colours_tech.war /home/ec2-user/apache-tomcat-9.0.78/webapps/"'
                }
            }
        }
    }
}


