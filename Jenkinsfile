node() {
    def docker = tool(name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool') + '/bin/docker'
    def test_passed = false
    def TEST_STRING_1 = '"messageId":1, "timestamp":1234, "protocolVersion":"1.0.0", "messageData":{"mMX":1234, "mPermGen":1234}'
    def TEST_STRING_2 = '"messageId":2, "timestamp":2234, "protocolVersion":"1.0.1", "messageData":{"mMX":1234, "mPermGen":5678, "mOldGen":22222}'
    def TEST_STRING_3 = '"messageId":3, "timestamp":3234, "protocolVersion":"2.0.0", "payload":{"mMX":1234, "mPermGen":5678, "mOldGen":22222, "mYoungGen":333333}'
    def TEST_RESPONSE_1 = 'Message: id=1, timestamp=1234, protocolVersion=1.0.0, messageData: mMx=1234, mPermGen=1234, mOldGen=null, mYoungGen=null'
    def TEST_RESPONSE_2 = 'Message: id=2, timestamp=2234, protocolVersion=1.0.1, messageData: mMx=1234, mPermGen=5678, mOldGen=22222, mYoungGen=null'
    def TEST_RESPONSE_3 = 'Message: id=3, timestamp=3234, protocolVersion=2.0.0, messageData: mMx=1234, mPermGen=5678, mOldGen=22222, mYoungGen=333333'
    
    stage ('pre-flight'){
        //cloning repo
        checkout([$class: 'GitSCM', branches: [[name: '*/kirillr']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'e781827d-8c1d-4cca-95f1-40c13a8e84d7', url: 'https://gitlab.com/nikolyabb/cs-devops2']]])
        prettyprintln("Cleaning up, expect much docker errors...")
        //cleaning up existing containers in case something failed last time
        try{
        sh "${docker} kill rabbit processor gateway"
        } catch (Exception ex){}

        try{
        sh "${docker} rm rabbit processor gateway"
        } catch (Exception ex){}

        try{
        sh "${docker} network disconnect messaging jen"    
        } catch (Exception ex){}

        try{
        sh "${docker} network rm messaging"    
        } catch (Exception ex){}
        
    }

    stage('build'){
        //maven build
        prettyprintln("Building maven package...")
        def maven = tool(name: 'maven', type: 'maven') + '/bin/mvn'
        sh "${maven} clean package"
        //build docker images
        prettyprintln("Building docker images...")
        sh "${docker} build -t processor message-processor"
        sh "${docker} build -t gateway message-gateway"
    }

    stage('deploy') {
        //run containers dunno why --networks must be that way probly magic
        prettyprintln("Deploying to localhost...")
        sh "${docker} network create -d bridge messaging"
        sh "${docker} run -d --rm --name rabbit --network='messaging' --hostname messageGatewayQueue -p 5672:5672 rabbitmq"
        sleep(60)
        sh "${docker} run -d --rm --name processor --network='container:rabbit' message-processor"
        sh "${docker} run -d --rm --name gateway  --network='messaging' message-gateway:latest"
        sh "${docker} ps -a"
        sleep(15)
    }

    stage('test'){
        //checking and cleanup
        prettyprintln("Checking if everything is all right...")
        sh "${docker} network connect messaging jen"
        sleep(10)

        def res1 = verifyViaTestString(TEST_STRING_1, TEST_RESPONSE_1)
        def res2 = verifyViaTestString(TEST_STRING_2, TEST_RESPONSE_2)
        def res3 = verifyViaTestString(TEST_STRING_3, TEST_RESPONSE_3)
        if (res1 && res2 && res3) {
            test_passed = true
            prettyprintln("All test messages passed")
        } else {
            prettyprintln("Tests failed")
            if(!res1){
                println("Problems with first message")
            }
            if(!res2){
                println("Problems with second message")                
            }
            if(!res3){
                println("Problems with third message")
            }
            test_passed = false
        }
        sh "${docker} kill rabbit processor gateway"
        sh "${docker} network disconnect messaging jen"    
        sh "${docker} network rm messaging"
    }

    stage('artifacts'){
        //upload artifacts to dockerhub if all is good
        if(test_passed){
            prettyprintln("Uploading artifacts...")
            withCredentials([usernamePassword(credentialsId: 'd4b6ac81-2fd2-49d2-8b0b-eb2a97db386f', passwordVariable: 'pass', usernameVariable: 'user')]) {
                sh "${docker} login -u${user} -p${pass}"
            }
            sh "${docker} tag gateway kradzanovskiy/gateway"
            sh "${docker} tag processor kradzanovskiy/processor"
            sh "${docker} push kradzanovskiy/gateway"
            sh "${docker} push kradzanovskiy/processor"
        } else {
            prettyprintln("Build encountered some errors")
        }
    }
}

def verifyViaTestString(queryString, respString) {
    def docker = tool(name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool') + '/bin/docker'
    def response = httpRequest(
        httpMode: 'POST', 
        requestBody: "{"+queryString+"}", 
        responseHandle: 'NONE', 
        url: "http://gateway:8080/message" )
    sleep 3
    outString = sh (
                script: "${docker} logs --tail 1 processor",
                returnStdout: true
                ).trim()
    println("REQUEST:$queryString")
    println("RESPONSE:$outString")
    println("EXPECTED_RESPONSE:$respString")
    if (respString == outString) {
        println("RESPONSE AND EXPECTED_RESPONSE MATCH")
        return true
    } else {
        println("RESPONSE AND EXPECTED_RESPONSE DIFFER")
        return false
    }
}
def prettyprintln(message){
    def sep = "--------------------------------"
    println(sep + message + sep)
}