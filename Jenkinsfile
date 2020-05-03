def binNumber = 15;
def binURL = 'https://google.com'

node() {
    stage('test'){
        sh 'env'
        
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


        //pre-requisites

        //download git repo
        git clone ...
        //build
        docker network create -d bridge messaging
        docker build -t message-processor message-processor
        docker build -t message-gateway message-gateway


        //creating containers
        docker run  --rm -d --hostname messageGatewayQueue --name rabbit --network='messaging' -p 5672:5672 rabbitmq
        docker run   -d --network='container:rabbit' --name processor -d message-processor
        docker run --name gateway -d --rm -p 8080:8080 --network='messaging' message-gateway:latest
    

        //checking
        curl http://localhost:8080/message -X POST -d '{"messageId":1, "timestamp":1234, "protocolVersion":"1.0.0", "messageData":{"mMX":1234, "mPermGen":1234}}'
    }
}