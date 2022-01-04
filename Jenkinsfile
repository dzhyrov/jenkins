node('master') {
    currentBuild.displayName = "${BUILD_ID} ${branch_name}"
    try {
        stage("upload") {
            pipeline = load 'file-workaround.groovy'
            def file_in_workspace = pipeline.call("platform_current")
            sh "cat ${file_in_workspace}"
        }
        stage('Prepare'){
            dir('pyezml'){
                checkout([$class: 'GitSCM', branches: [[name: '*/${branch_name}']], userRemoteConfigs: [[url: '/var/jenkins_home/repo/pyezml.git']]])
                sh '''
                pwd
                rm -f images/*.png
                rm -f *.log
                if ! [ -f ./venv ];then
                    python3 -m venv venv
                fi
                . venv/bin/activate
                pip --proxy http://web-proxy.corp.hpecorp.net:8080 install -r requirements.txt
                '''
            }
        }
        stage('Run'){
            dir('pyezml'){
                    sh '''
                    . venv/bin/activate
                    export LOCAL_DRIVER=False
                    pytest -x -v -s --durations=0 --config platform_current.json ${test_script} --junitxml=junit_report.xml
                    '''
            }
        }
    } catch (e){
        throw e
    } finally {
        stage('Report'){
            junit 'pyezml/junit_report.xml'
            archiveArtifacts artifacts: 'pyezml/images/**/*.png, pyezml/*.log', fingerprint: true, allowEmptyArchive: true
        }
    }
}

