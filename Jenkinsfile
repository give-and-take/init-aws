pipeline {

    agent any

    environment {
        CI = true
    }

    stages {

        stage('Plan resource groups') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        sh """
                            cd resourcegroup
                            terraform init -backend-config='access_key=$USER' -backend-config='secret_key=$PASS' -backend-config='bucket=${env.MY_APP}-terraform'
                            terraform plan -no-color -out=tfplan -var 'access_key=$USER' -var 'secret_key=$PASS'
                        """
                    }
                }
            }
        }

        stage('Plan network') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        sh """
                            cd network
                            terraform init -backend-config='access_key=$USER' -backend-config='secret_key=$PASS' -backend-config='bucket=${env.MY_APP}-terraform'
                            terraform plan -no-color -out=tfplan -var 'access_key=$USER' -var 'secret_key=$PASS' -var 'basename=${env.BASENAME}'
                        """
                    }
                }
            }
        }

        stage('Plan ACM certificate') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        sh """
                            cd dns_certificate
                            terraform init -backend-config='access_key=$USER' -backend-config='secret_key=$PASS' -backend-config='bucket=${env.MY_APP}-terraform'
                            terraform plan -no-color -out=tfplan -var 'access_key=$USER' -var 'secret_key=$PASS' -var 'main_domain=${env.MY_MAIN_DOMAIN}' -var 'domain=${env.MY_DOMAIN}'
                        """
                    }
                }
            }
        }

        /*
        stage('Plan users') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        sh """
                            cd users
                            terraform init -backend-config='access_key=$USER' -backend-config='secret_key=$PASS' -backend-config='bucket=${env.MY_APP}-terraform'
                            terraform plan -no-color -out=tfplan -var 'access_key=$USER' -var 'secret_key=$PASS' -var 'console_user=andre_nho' -var 'main_domain=${env.MY_MAIN_DOMAIN}'
                        """
                        // sh "echo Add the user/pass credentials above to Jenkins with the id 'dynamo.'"
                    }
                }
            }
        }
        */

        stage('Deploy changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                        if (env.BRANCH_NAME == "master") {
                            timeout(time: 10, unit: 'MINUTES') {
                                input(id: "Deploy Gate", message: "Deploy application?", ok: 'Deploy')
                            }
                        }
                        sh """
                            cd resourcegroup   && terraform apply -no-color -lock=false -input=false tfplan && cd ..
                            cd network         && terraform apply -no-color -lock=false -input=false tfplan && cd ..
                            cd dns_certificate && terraform apply -no-color -lock=false -input=false tfplan && cd ..
                        """
                    }
                }
            }
        }

    }
}

// vim:st=4:sts=4:sw=4:expandtab:syntax=groovy
