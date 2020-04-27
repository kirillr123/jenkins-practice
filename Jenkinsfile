def binNumber = 15;
def binURL = 'https://google.com'

node() {
    stage('test'){
        sh 'env'
        echo 'Hello World'
        
        def textFild = """ Long text
        and more text...
        and more
        """.trim()
    }
    stage('scm') {
        sh 'ls -la'
        sh 'pwd'
    }
    stage('build') {
        def docker = tool(name: 'dockerLatest', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool') + '/bin/docker'
        
        sh "ls -la ${docker}" 
        sh "${docker} ps -a"
        
    }
}
