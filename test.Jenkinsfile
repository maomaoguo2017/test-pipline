properties([
    parameters([
        string(name: 'countTotal', defaultValue: '4'),
        choice(name: 'servers', choices: ['10.10.10.10'], description: 'Run on specific platform'),
    ])
])

node{
    stage('common task'){

        sh "echo common task"

    }
    def stages = [failFast: true]
    for (int i = 1; i < params.countTotal.toInteger()+1; i++) {

        def vmNumber = i
        stages["server${vmNumber}"] = {
            stage("deploy ${vmNumber}") {

                    sh "echo deploy; sleep 5"

            }
            stage("test ${vmNumber}") {

                    sh "echo testing; sleep 5"

            }
        }
        
    }
    parallel stages
}