pipeline {
    agent any

    parameters {
        string(name: 'environment', defaultValue: 'terraform', description: 'Workspace/environment file to use for deployment')
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        booleanParam(name: 'destroy', defaultValue: false, description: 'Destroy Terraform build?')

    }


    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage ('Actualizacion repo'){
            steps{
                script{
                    cleanWs()
                    checkout([$class: 'GitSCM',
                        branches: [[name: "developer"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [],
                        submoduleCfg: [],
                        userRemoteConfigs: [[credentialsId: "VantiBitbucket", url: "https://bitbucket.org/ed01406/vanti_repository" ]]
                    ])
                }
            }
        }
// ------------------------------------
// -- ETAPA: Checkov
// ------------------------------------
        stage('Test') {
            steps {
                script {
                    try {
                        sh "checkov --directory /var/lib/jenkins/workspace/grupo_vanti_developer_2/terraform/modules -o junitxml > result.xml || true"
                        junit "result.xml"
                        echo "Hola"
                    } catch (err) {
                        echo "Hola catch"
                        if (currentBuild.result == 'UNSTABLE')
                            currentBuild.result = 'FAILURE'
                        throw err
                    }
                }
            }
        }
// ------------------------------------
// -- ETAPA: Terraform
// ------------------------------------
        stage('Plan') {
            when {
                not {
                    equals expected: true, actual: params.destroy
                }
            }
            
            steps {
                sh 'cd terraform/modules/${environment}; terraform init -reconfigure'
                sh 'pwd'
                sh 'cd terraform/modules/${environment}; terraform plan -input=false -out tfplan'
                sh 'cd terraform/modules/${environment}; terraform show -no-color tfplan > /var/lib/jenkins/workspace/grupo_vanti_developer_2/tfplan.txt'

            }
        }
        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
               not {
                    equals expected: true, actual: params.destroy
                }
           }
           
                
            

           steps {
               script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            when {
                not {
                    equals expected: true, actual: params.destroy
                }
            }
            
            steps {
                sh 'cd terraform/modules/${environment}; terraform apply -input=false tfplan'
            }
        }
        
        stage('Destroy') {
            when {
                equals expected: true, actual: params.destroy
            }
        
        steps {
           sh ('cd terraform/modules/${environment}; terraform destroy --auto-approve')
        }
    }
    }
    options {
        preserveStashes()
        timestamps()
        ansiColor('xterm')
    }
}

