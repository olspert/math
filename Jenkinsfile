pipeline {
    agent any
    stages {
        stage('Clean & Setup') {
            steps {
                sh 'git clean -xffd'
                sh "echo 'CC=${env.CXX}' > make/local"
                sh "echo 'CXXFLAGS += -Werror' >> make/local"
            }
        }
        stage('Linting checks') {
            steps {
                parallel(
                    cppLint: { sh "make cpplint" },
                    dependencies: { sh 'make test-math-dependencies' },
                )
            }
        }
        stage('Documentation') {
            steps { sh 'make doxygen' }
        }
        stage('Headers') {
            steps { sh "make -j${env.PARALLEL} test-headers" }
        }
        stage('Unit') {
            steps { sh "./runTests.py -j${env.PARALLEL} test/unit" }
        }
        stage('Distribution tests') {
            agent { label "distribution" }
            steps { sh './runTests.py -j${env.PARALLEL} test/prob' }
        }
        stage('Update Stan Upstream') {
            steps {
                sh "curl -O https://raw.githubusercontent.com/stan-dev/ci-scripts/master/jenkins/create-stan-pull-request.sh"
                sh "sh create-stan-pull-request.sh"
            }
        }
    }
    post {
        always {
            warnings consoleParsers: [[parserName: 'GNU C Compiler 4 (gcc)']], failedTotalAll: '0', usePreviousBuildAsReference: false, canRunOnFailed: true
            warnings consoleParsers: [[parserName: 'Clang (LLVM based)']], failedTotalAll: '0', usePreviousBuildAsReference: false, canRunOnFailed: true
            warnings consoleParsers: [[parserName: 'CppLint']], failedTotalAll: '0', usePreviousBuildAsReference: false, canRunOnFailed: true
            warnings consoleParsers: [[parserName: 'math-dependencies']], failedTotalAll: '0', usePreviousBuildAsReference: false, canRunOnFailed: true
        }
    }
}
