node {
    stage('Install') {
        git 'https://github.com/wantotri/simple-python-pyinstaller-app.git'
    }

    stage('Build') {
        docker.image('python:2-alpine').inside {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }

    stage('Test') {
        docker.image('qnib/pytest').inside {
            try {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            } finally {
                junit 'test-reports/results.xml'
            }
        }
    }

    stage('Manual Approval') {
        input message: 'Lanjut ke tahap Deploy?'
    }

    stage('Deploy') {
        def VOLUME = '$(pwd)/sources:/src'
        def IMAGE = 'cdrx/pyinstaller-linux:python2'

        dir(path: env.BUILD_ID) {
            unstash(name: 'compiled-results')
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
        }

        archiveArtifacts artifacts: "${env.BUILD_ID}/sources/dist/add2vals", fingerprint: true
        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"

        withCredentials([sshUserPrivateKey(credentialsId: 'aws-wantotrees-key', keyFileVariable: 'secret_file', usernameVariable: 'ssh_user')]) {
            sh "scp -o StrictHostKeyChecking=no -i ${secret_file} ${env.BUILD_ID}/sources/dist/add2vals ${ssh_user}@18.136.176.208:/home/${ssh_user}/dicoding/"
        }

        sleep(time:1, unit:"MINUTES")
    }
}
