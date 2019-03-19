node ('master') {
    def uniqueId = UUID.randomUUID().toString()
    def imageName = 'ubuntu' //can be any image you want to execute

    //Writing a file to demonstrate how to copy (stash/unstash) files to the container
    sh 'echo Hello World > test.txt'
    stash 'masterWorkspace'

    podTemplate(
        cloud: 'kubernetes',
        name: 'test-k8s-jenkins',
        label: uniqueId, 
        containers: [
            containerTemplate(
                name: 'jnlp', 
                image: 'alxsap/temp:jenkinsci-docker-jnlp-slave-before',
                args: '${computer.jnlpmac} ${computer.name}'
            ),

            containerTemplate (
                name: 'container-exec', 
                image: imageName,
                command: '/usr/bin/tail -f /dev/null',
            )
        ]
    )
    {
        node (uniqueId) {

            echo "Print JNLP env"
            container('jnlp'){
                sh '''
                    env
                    ls -la /home/jenkins/.jenkins
                    ls -la /home/jenkins/agent
                '''
            }


            echo "Execute container content in Kubernetes pod"        
            container('container-exec'){
                // unstash content from Jenkins master workspace
                unstash 'masterWorkspace'
                sh '''
                    cat test.txt
                '''
            }
            // stash results
            stash 'containerResults'
        }
    }
    // unstash to Jenkins master workspace      
    unstash 'containerResults'
}
