pipeline {
    agent none
    environment {
      SOURCE_DIR = '/data/pipeline_demo'
    }
    triggers {
        GenericTrigger (
            causeString: 'Triggered', 
            genericVariables: [
              [key: 'ref', value: '$.ref'],
              [key: 'action', value: '$.action'],
              [key: 'merge_commit', value: '$.pull_request.merge_commit_sha'],
              [key: 'branch', value: '$.workflow_run.head_branch'],
              [key: 'repo', value: '$.repository.name'],
              [key: 'pull_request_title', value: '$.pull_request.title'],
              [key: 'result', value: '$.workflow_run.conclusion']
            ], 
            printContributedVariables: true, 
            printPostContent: true,
            regexpFilterExpression: 'completed\\sdevelop\\ssuccess',
            regexpFilterText: '$action $branch $result',
            token: 'nova'
        )
    }

    stages {
        
        stage('Deploy to test'){
            when {
                branch 'master'
            }
            parallel {
               
                stage('pull ${module}') {
                    agent {
                      node {
                        label "dingo_stack"
                      }
                    }
            
                    steps {
                        echo "pull ${module} images to test"
                        sh 'kolla-ansible -i /root/multinode pull --tag ${module} -e openstack_tag=latest'
                        echo 'deploy images to develop '
                        sh 'kolla-ansible -i /root/multinode upgrade --tag ${module} -e openstack_tag=latest'
                    }
                }
            }
        }
        stage('deploy dingoOps to dev'){
            when {
                anyOf { branch 'develop'; branch 'stable/2023.2' }
            }

            parallel {
              
                stage('pull ${module}') {
                    agent {
                      node {
                        label "dingo_stack"
                      }
                    }
                    steps {
                        echo "pull ${module} images to dev"
                        sh 'kolla-ansible -i /root/multinode pull --tag ${module} -e openstack_tag=${branch}'
                        echo 'deploy images to develop '
                        sh 'kolla-ansible -i /root/multinode upgrade --tag ${module} -e openstack_tag=${branch}'
                    }
                }
            }
        }
    }
}
