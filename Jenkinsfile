pipeline {
    agent any
    
    stages {
        stage('Install and Configure Puppet Agent') {
            steps {
                script {
                    // Step 1: Install Puppet Agent on the slave node using SSH key
                    sh 'sudo ssh -i /root/.ssh/id_rsa ec2-user@44.217.5.12 "curl -O https://yum.puppet.com/puppet7-release-el-7.noarch.rpm"'
                    sh 'sudo ssh -i /root/.ssh/id_rsa ec2-user@44.217.5.12 "sudo yum install -y puppet-agent"'
                    
                    // Step 2: Configure Puppet Agent to connect to the Puppet Master using SSH key
                    sh 'sudo ssh -i /root/.ssh/id_rsa ec2-user@44.217.5.12 "sudo /opt/puppetlabs/bin/puppet config set server 23.20.131.69"'
                    
                    // Step 3: Test Puppet Agent configuration using SSH key
                    sh 'sudo ssh -i /root/.ssh/id_rsa ec2-user@44.217.5.12 "sudo /opt/puppetlabs/bin/puppet agent -t"'
                }
            }
        }
        
        stage('Push Ansible Configuration to Install Docker') {
            steps {
                script {
                    // Step 1: Copy Ansible playbook to remote server
                    sh 'sudo scp -i /root/.ssh/id_rsa /home/ec2-user/install_docker.yml ec2-user@44.217.5.12:/home/ec2-user/'
            
                    // Step 2: Install Ansible on the remote server
                    sh 'sudo ssh -i /root/.ssh/id_rsa ec2-user@44.217.5.12 "sudo amazon-linux-extras install ansible2"'
            
                    // Step 3: Run Ansible playbook to install Docker
                    sh 'sudo ssh -i /root/.ssh/id_rsa ec2-user@44.217.5.12 "ansible-playbook /home/ec2-user/install_docker.yml"'
                }
            }
        }
        
        stage('Build and Deploy PHP Docker Container') {
            steps {
                script {
                    def remoteHost = '44.217.5.12'
                    def sshKeyPath = '/root/.ssh/id_rsa'
                    def remoteDir = '/home/ec2-user/php_app'
                    def directoryExists

                    try {
                        directoryExists = sh(
                            returnStdout: true,
                            script: "sudo ssh -i $sshKeyPath ec2-user@$remoteHost '[ -d $remoteDir ] && echo true || echo false'"
                        ).trim()
                    } catch (Exception e) {
                        directoryExists = 'false'
                    }

                    if (directoryExists == 'true') {
                        sh "sudo ssh -i $sshKeyPath ec2-user@$remoteHost 'cd $remoteDir && git pull'"
                    } else {
                        sh "sudo ssh -i $sshKeyPath ec2-user@$remoteHost 'git clone https://github.com/Realsammy/projCert.git $remoteDir'"
                    }

                    sh """
                        sudo ssh -i $sshKeyPath ec2-user@$remoteHost '
                            containers_to_stop=\$(docker ps -q -f ancestor=php_app)
                            if [ -n "\$containers_to_stop" ]; then
                                sudo docker stop \$containers_to_stop
                            fi

                            containers_to_remove=\$(docker ps -aq -f status=exited)
                            if [ -n "\$containers_to_remove" ]; then
                                sudo docker rm \$containers_to_remove
                            fi

                            cd $remoteDir
                            sudo docker build -t php_app .
                            sudo docker run -d -p 80:80 php_app
                        '
                    """
                }
            }
        }
    }
}
