pipeline {
    agent any
    environment {
        registryURI = "registry.hub.docker.com/"
        dev_registry = "prakuldip/noori-anal-project-dev"
        qa_registry = "prakuldip/noori-anal-project-qa"
        stage_registry = "prakuldip/noori-anal-project-stage"
        prod_registry = "prakuldip/noori-anal-project-prod"
        dev_registryCredential = "dev-dh-cred"
        qa_registryCredential = "qa-dh-cred"
        commitid = "${params.commit_id}"
        
    }
    parameters {
            choice(name: 'account', choices: ['dev', 'qa', 'stage', 'prod'], description: 'Select the environment')
            string(name: 'commit_id', defaultValue: "${GIT_COMMIT}", description: 'provide commit id if specific.')
        }
stages {
        stage('build and push docker image in dev repo') {
            when {
                    expression {
                        params.account == 'dev'
                    }
            }
            environment{
              DEST_IMAGE = "${registryURI}${dev_registry}:${commitid}"
              DST_REGISTRY_CRED = "${dev_registryCredential}"
            }
            steps{
                script {
                    docker.withRegistry("https://${env.registryURI}",DST_REGISTRY_CRED){
                    docker.build(DEST_IMAGE).push()
                    }
                    echo "Removing the image locally"
                    sh "docker image rm '${DEST_IMAGE}'"
                }
            }  
        }
        stage('pull from dev and push in to qa repo') {
            when {
                    expression {
                        params.account == 'qa'
                    }
            }
            environment{
              SRC_IMAGE  = "${registryURI}${dev_registry}:${commitid}"
              DEST_IMAGE = "${registryURI}${qa_registry}:${commitid}"
              SRC_REGISTRY_CRED = "${dev_registryCredential}"
              DST_REGISTRY_CRED = "${qa_registryCredential}"
            }
            steps{
              imagePullTagPush(SRC_IMAGE,DEST_IMAGE,SRC_REGISTRY_CRED,DST_REGISTRY_CRED)
              echo "Removing images localy"
              sh "docker image rm '${SRC_IMAGE}' '${DEST_IMAGE}'"
            }  /* defining and calling function is successful*/
        }
    }
    post { 
        always { 
            echo 'Deleting Workspace'
            deleteDir() /* clean up our workspace */
        }
    }
}
def imagePullTagPush(String SRC_IMAGE, String DEST_IMAGE, String SRC_REGISTRY_CRED,String DST_REGISTRY_CRED) {
    def image_to_pull = docker.image(SRC_IMAGE)
    docker.withRegistry("https://${registryURI}",SRC_REGISTRY_CRED){
    image_to_pull.pull()
    }
    sh "docker image tag '${SRC_IMAGE}' '${DEST_IMAGE}'"
    def image_to_push = docker.image(DEST_IMAGE)
    docker.withRegistry("https://${registryURI}",DST_REGISTRY_CRED){
    image_to_push.push()
    }
} /* NOTE: when you are using same image registry, then you have to create a seperate token for each dev,qa,
stage,prod.... You can not use same token and create multiple credentials in jenkins. While creating
multiple credentials for same registry you have to created multiple tokens.*/
