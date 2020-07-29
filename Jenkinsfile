pipeline{
    agent any
    parameters{
        string (name: 'aws_key', description: '')
        password (name: 'aws_secret', description: '')
        string (name: 's3_name', description: 's3 bucket name to store and retrive terraform state file')
       // text (name: 'tools', description: "Pass the tools name and creds in list format eg [{'tools':'jenkins', 'u_name':'admin','password':'admin123'}]")
        
        string (name: 'jenkins_user', description: '')
        password (name: 'jenkins_password', description: '')
        string (name: 'sonar_user', description: '')
        password (name: 'sonar_password', description: '')
       /* booleanParam (name: 'jenkins', defaultValue: false,  description: 'check this box to install jenkins tool')
        booleanParam (name: 'git', defaultValue: false,  description: 'check this box to install git tool')
        booleanParam (name: 'sonar', defaultValue: false,  description: 'check this box to install sonar tool')
        booleanParam (name: 'nexus', defaultValue: false,  description: 'check this box to install nexus tool')*/
     }
    environment {
        AWS_ACCESS_KEY_ID = "${params.aws_key}"
        AWS_SECRET_ACCESS_KEY = "${params.aws_secret}"
    }
    /*-------- CODE CLONE ------------ */
    stages{
        stage('git pull'){
            steps{
           // echo "${params.jenkins}"
            echo "${params.tools}"
            checkout ([$class: 'GitSCM', branches: [[name: '*/master']],
                       userRemoteConfigs: [[credentialsId: 'git_key', url: 'URL']],
                ])
            }
        }
        /*--------------------------- Persistent Stage ----------------------- */
        stage('persistent_stage'){
            steps{
                script{
                 def tfHome = tool name:'Terraform'
                 env.PATH = "${tfHome}:${env.PATH}"
                 def ansHome = tool name:'ansible'
                 env.PATH = "${ansHome}:${env.PATH}"
                 sh 'rm -rf $WORKSPACE/echo_key'
                }
                sh '''
                  cd $WORKSPACE/terraform/vpc-subnet
                  terraform init -backend-config backend/s3-backend.tf -reconfigure
                  terraform apply -auto-approve
                  terraform output ssh_private > $WORKSPACE/echo_key
                  '''
                script{
                       sh 'chmod 400 $WORKSPACE/echo_key'
                     }
             }
        }
        /*    ----------------- Jenkins Stage ------------------- */
      /*  stage('jenkins'){
            steps{
                 sh '''
                   cd $WORKSPACE/terraform/jenkins
                   terraform init -backend-config backend/s3-backend.tf -reconfigure
                   terraform apply -auto-approve
                   terraform output public_ip > jenkins_public_ip
                 '''
                 script{
                       env.jenkins_public_ip = readFile('terraform/jenkins/jenkins_public_ip').trim()
                       echo "${jenkins_public_ip}"
                 }
                 sleep(time:60,unit:"SECONDS")
                 dir('ansible')
                 {
                  sh 'sed -i "s/<JENKINS_IP>/"${jenkins_public_ip}"/g" host'
                  sh '''
                        sed -i "s/<JENKINS_USER>/"${jenkins_user}"/g" roles/jenkins/vars/main.yml
                        sed -i "s/<JENKINS_PASSWORD>/"${jenkins_password}"/g" roles/jenkins/vars/main.yml
                        export ANSIBLE_HOST_KEY_CHECKING=False
                        ansible-playbook -i host jenkins.yaml --private-key ../echo_key --ssh-common-args="-oStrictHostKeyChecking=no"
                  '''
                 }
              }
       } */ 
       /*---------------- Jenkins END ------------------*/
        /*    ----------------- Sonar Stage ------------------- */
      /* stage('sonar'){
           steps{
               dir('terraform/sonarqube')
               {
                sh '''
                   terraform init -backend-config backend/s3-backend.tf -reconfigure
                   terraform apply -auto-approve
                   terraform output public_ip > sonar_public_ip
                 '''
               }
                script{
                      env.sonar_public_ip = readFile('terraform/sonarqube/sonar_public_ip').trim()
                }
                sleep(time:60,unit:"SECONDS")
                dir('ansible')
                {
                 sh 'sed -i "s/<SONAR_IP>/"${sonar_public_ip}"/g" host'
                 sh '''
                       sed -i "s/<SONAR_USER>/"${sonar_user}"/g" roles/sonarqube/vars/main.yml
                       sed -i "s/<SONAR_PASSWORD>/"${sonar_password}"/g" roles/sonarqube/vars/main.yml
                       export ANSIBLE_HOST_KEY_CHECKING=False
                       ansible-playbook -i host sonarqube.yaml --private-key ../echo_key --ssh-common-args="-oStrictHostKeyChecking=no"
                     '''
                }
             }
      } */
      /*---------------- Sonar END ------------------*/
      /*    ----------------- Gitlab Stage ------------------- */
         stage('gitlab'){
             steps{
                 script {
                         env.gitlab_ip = terraform('gitlab','gitlab')
                         echo "${gitlab_ip}"
                 }
               }
        }
        /*---------------- gitlab END ------------------*/



     }
  }
 


def terraform(path,component){
       dir (terraform/"${path}") {
           sh '''
                terraform init -backend-config backend/s3-backend.tf -reconfigure
                terraform apply -auto-approve
                terraform output public_ip > public_ip
              '''
           script{
                        env.public_ip = readFile('terraform/"${path}"/public_ip').trim()
                  }
           return public_ip
    }
}
       
       
