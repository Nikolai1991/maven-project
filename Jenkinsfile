pipeline {
    agent any
    tools {
            maven 'localMaven'
            jdk 'localJDK'
    }
    parameters {
         string(name: 'tomcat_dev', defaultValue: '54.172.112.30', description: 'Staging Server')
         string(name: 'tomcat_prod', defaultValue: '34.230.40.123', description: 'Production Server')
    }

    triggers {
         pollSCM('* * * * *')
     }

stages{
        stage ('Initialize') {
                steps {
                    sh '''
                        echo "PATH = ${PATH}"
                        echo "M2_HOME = ${M2_HOME}"
                    '''
                }
        }
        stage('Build'){
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage ('Deployments'){
            parallel{
                stage ('Deploy to Staging'){
                    steps {
                        sh "ssh -i "jenkins.pem" ec2-user@ec2-54-172-112-30.compute-1.amazonaws.com"
                        sh "chown ec2-user /var/lib/tomcat7/webapps"
                        sh "scp -i /home/ec2-user/key/jenkins.pem **/target/*.war ec2-user@${params.tomcat_dev}:/var/lib/tomcat7/webapps"
                    }
                }

                stage ("Deploy to Production"){
                    steps { 
                        sh "scp -i /home/ec2-user/key/jenkins.pem **/target/*.war ec2-user@${params.tomcat_prod}:/var/lib/tomcat7/webapps"
                    }
                }
            }
        }
    }
}
