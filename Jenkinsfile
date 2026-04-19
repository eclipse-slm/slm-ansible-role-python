def scenarios = [
    "ubuntu2404",
    "ubuntu2204",
    "ubuntu2004",
    "ubuntu1804",
    "centos10",
    "centos9",
    "centos8",
    "centos7",
    "debian13",
    "debian12",
    "debian11"
]

def verbose = "-vv"

parallel_stages = [:]

for (s in scenarios) {
    def scenario = s

    parallel_stages[scenario] = {
        docker
          .image("${MOLECULE_DOCKER_IMAGE}")
          .inside("--name ${JOB_NAME}_${scenario} -e OS_AUTH_URL=${OS_AUTH_URL} -e OS_USERNAME=${OS_APPLICATION_CREDENTIAL_ID} -e OS_PASSWORD=${OS_APPLICATION_CREDENTIAL_SECRET} -u root") {

            try {
                stage("Test") {
                    sh "molecule ${verbose} test -s ${scenario} --destroy never --report"
                }
            } finally {                
                stage("Destroy") {
                    sh "molecule destroy -s ${scenario} --report"
                }
            }
        }
    }
}

node {
   checkout scm
    withCredentials([usernamePassword(
            credentialsId: 'openstack-credentials',
            usernameVariable: 'OS_APPLICATION_CREDENTIAL_ID',
            passwordVariable: 'OS_APPLICATION_CREDENTIAL_SECRET'
    )]) {
        stage("Pull molecule image") {
            sh "docker pull ${MOLECULE_DOCKER_IMAGE}"
        }

        parallel(parallel_stages)
    }
}
