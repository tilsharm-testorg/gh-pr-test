def fullBranchUrl(branchName) { return "${scm.getUserRemoteConfigs()[0].getUrl()}/tree/$branchName" }

def getBranchName() { 
    if(env.CHANGE_ID != null) {
        return 'test1'
    } else {
        return 'master'
    }

}

def getGitUrl() {
    if(env.CHANGE_ID != null) {
        return 'https://github.com/tilsharm-testorg/gh-pr-test.git'
    } else {
        return 'https://github.com/TilakShrma/gh-pr-test.git'
    }
}

def sampleComment = '''
    |----Metrics-----|----BaseLine----|----PR-----|----Delta----|
    |Skipped Test    | ${sampleBaseLine}|      NA   |      NA     |
    |Failed Test     |   NA           |      NA   |      NA     |
    |Total Test      |   NA           |      NA   |      NA     |
    |Line Coverage % |   NA           |      NA   |      NA     |
    |uncovered lines |   NA           |      NA   |      NA     |
    |Total Lines     |   NA           |      NA   |      NA     |
    |----------------|----------------|-----------|-------------|
'''

timestamps {
    node(label: 'master') {
        stage('Checkout Git Repo') {
            git credentialsId: 'fe4effdc-f62d-4624-bcc7-d4749675f873',
            branch: getBranchName(),
            url: getGitUrl()
        }
        stage('Archive and Record Tests') {
            if (fileExists('output/coverage/jest/cobertura-coverage.xml') && fileExists('output/coverage/jest/jest-junit.xml')) {
                archiveArtifacts 'output/coverage/jest/cobertura-coverage.xml'
                archiveArtifacts 'output/coverage/jest/jest-junit.xml'
                cobertura coberturaReportFile: 'output/coverage/jest/cobertura-coverage.xml'
            }
            else {
                echo 'XML report were not created'
            }
        }
        stage('Copy artifacts from master'){
            if(env.CHANGE_ID != null){
                copyArtifacts filter: 'output/', projectName: 'master', selector: lastCompleted(), target: 'master/'
                bat "C:/Python27/python.exe ./bin/xmlToJson.py master/output/coverage/jest/cobertura-coverage.xml --type=cobertura"
                bat "C:/Python27/python.exe ./bin/xmlToJson.py master/output/coverage/jest/jest-junit.xml --type=jest"
                bat "C:/Python27/python.exe ./bin/xmlToJson.py output/coverage/jest/cobertura-coverage.xml --type=cobertura"
                bat "C:/Python27/python.exe ./bin/xmlToJson.py output/coverage/jest/jest-junit.xml --type=jest"
            }
        }
        stage('Generate comparision metrics'){
            if(fileExists('pr-coverage-report.json') && fileExists('master-coverage-report.json')){
                echo "coverage report found for master and pr"
                def result = bat (script: "bin/prComparisonMetrics.py master-coverage-report.json pr-coverage-report.json", returnStdout: true)
                echo result
            }
        }
        stage('Record Coverage') {
            if (env.CHANGE_ID != null) {
            currentBuild.result = 'SUCCESS'
            step([$class: 'MasterCoverageAction', scmVars: [GIT_URL: 'https://github.com/TilakShrma/gh-pr-test.git']])
            } 
            else if (env.CHANGE_ID != null) {
            currentBuild.result = 'SUCCESS'
            step([$class: 'CompareCoverageAction', publishResultAs: 'statusCheck', scmVars: [GIT_URL: 'https://github.com/TilakShrma/gh-pr-test.git']])
            pullRequest.comment(sampleComment)
        }
            
        }
        stage('Clean Workspace') {
            cleanWs notFailBuild: true
        }
    }
}

