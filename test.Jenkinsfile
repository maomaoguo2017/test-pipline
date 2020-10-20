def getBranch(){
    return "master"
}
def getServers(){
    return ['10.10.10.10']
}
pipeline {
    agent any
    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 60, unit: 'MINUTES')
        buildDiscarder logRotator(numToKeepStr: '30')
    }

    environment {
        APP_NAME = "test_name"
    }

        stage('确认发版信息') {
            input {
                message '请确认「代码分支」'
                ok '确认'
                parameters {
                    string defaultValue: 'master', description: '默认分支为「master」，可修改为 tag 名称：', name: 'BRANCH_NAME', trim: true
                }
            }
            steps {
                script {
                    env.BRANCH_NAME = "${BRANCH_NAME}"
                }
                input message: "请确认发版信息: \n项目：${APP_NAME} \n代码分支：${BRANCH_NAME} \n运行环境：${DEPLOY_ENVIRONMENT}", ok: '确认'
            }
        }
        stage('初始化环境') {
            // Each stage is made up of steps
            steps {
                script {
                    SERVERS = getServers()
                    GIT_BRANCH = getBranch()
                    echo "git branch: ${GIT_BRANCH}"
                }
            }                
        }
        stage('部署全部服务器') {
            environment {
                MAIN_SERVER = "${env.MAIN_SERVER}"
                HEALTH_CHECK_URL = "http://${MAIN_SERVER}/${env.APP_NAME}"
            }
        }
        def stages = [failFast: true]
        for (int i = 1; i < SERVERS.size(); i++){
            def vmNumber = i
            stages["server${vmNumber}"] = {
                SERVER = "${SERVERS[i]}"
                echo "部署服务器：${SERVER}"
                sleep 20

                echo "验证服务：curl -sL -w '%{response_code}' ${HEALTH_CHECK_URL}"
                script {
                    def (String response, int code) = sh(script: "curl -sL -w '%{response_code}' ${HEALTH_CHECK_URL}", returnStdout: true).trim().tokenize("\n")
                    echo "code: ${code},response: ${response}"
                    if (code != 200 || response != "${HEALTH_CHECK_RESPONSE}") {
                        unstable "服务器 ${SERVER} 验证失败，状态码：${code}，响应：${response}"
                    }
                }
            }
        }
        parallel stages
    
}
