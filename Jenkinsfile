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
            string(name: 'commit_id', defaultValue: 'latest', description: 'provide commit id if specific')
        }
stages {
        stage('build and push docker image in dev repo') {
            environment{
              DEST_IMAGE = "${registryURI}${dev_registry}:${commitid}"
              DST_REGISTRY_CRED = "${dev_registryCredential}"
            }
            steps{
                script {
                    docker.withRegistry("https://${env.registryURI}",DST_REGISTRY_CRED){
                    docker.build("${DEST_IMAGE}:${commitid}").push()
                    }
                    sh "docker image rm '${registryURI}${registry}:$GIT_COMMIT'"
                }
            }  
        }
        stage('pull from dev and push in to qa repo') {
            environment{
              SRC_IMAGE  = "${registryURI}${dev_registry}:${commitid}"
              DEST_IMAGE = "${registryURI}${qa_registry}:${commitid}"
              SRC_REGISTRY_CRED = "${dev_registryCredential}"
              DST_REGISTRY_CRED = "${qa_registryCredential}"
            }
            steps{
              imagePullTagPush(SRC_IMAGE,DEST_IMAGE,SRC_REGISTRY_CRED,DST_REGISTRY_CRED)
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
    sh "docker image tag '${registryURI}${dev_registry}:${imageTag}' '${registryURI}${qa_registry}:${imageTag}'"
    def image_to_push = docker.image(DEST_IMAGE)
    docker.withRegistry("https://${registryURI}",DST_REGISTRY_CRED){
    image_to_push.push()
    }
}