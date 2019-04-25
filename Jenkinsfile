podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.02',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'ibmcom/k8s-helm:v2.6.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        def branch
        stage ('Extract') {
            echo 'Pulling...' + env.BRANCH_NAME
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
            branch = env.BRANCH_NAME
        }
        def repository
        stage ('Docker') {
            container ('docker') {
                def registryIp = sh(script: 'getent hosts registry.kube-system | awk \'{ print $1 ; exit }\'', returnStdout: true).trim()
                repository = "${registryIp}:80/${branch}"
                sh "docker build -t ${repository}:${commitId} ."
                sh "docker push ${repository}:${commitId}"
            }
        }
        stage ('Deploy') {
            container ('helm') {
                sh "/helm init --client-only --skip-refresh"
				sh "ls -l"
                sh "/helm upgrade --install --set image.repository=${repository},image.tag=${commitId},image.branch=${branch} ${branch} ."
            }
        }
		stage ('Delete unused images') {
            container ('docker') {
                sh "docker images -q |xargs docker rmi --force"
            }
        }
    }
}