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

def comment = pullRequest.comment('This PR comment is generated by jenkins');

timestamps {
    node(label: 'master') {
        stage('Checkout Git Repo') {
            git credentialsId: 'fe4effdc-f62d-4624-bcc7-d4749675f873',
            branch: getBranchName(),
            url: getGitUrl()
        }
        stage('Archive and Record Tests') {
            if (fileExists('output/coverage/jest/cobertura-coverage.xml')) {
                archiveArtifacts 'output/coverage/jest/cobertura-coverage.xml'
                cobertura coberturaReportFile: 'output/coverage/jest/cobertura-coverage.xml'
            }
            else {
                echo 'XML report were not created'
            }
        }
        stage('Record Coverage') {
            if (env.CHANGE_ID == null) {
            currentBuild.result = 'SUCCESS'
            step([$class: 'MasterCoverageAction', scmVars: [GIT_URL: 'https://github.com/TilakShrma/gh-pr-test.git']])
            } 
            else if (env.CHANGE_ID != null) {
            currentBuild.result = 'SUCCESS'
            step([$class: 'CompareCoverageAction', publishResultAs: 'statusCheck', scmVars: [GIT_URL: 'https://github.com/TilakShrma/gh-pr-test.git']])
            comment.body('This PR looks good');
        }
            
        }
        stage('Clean Workspace') {
            cleanWs notFailBuild: true
        }
    }
}

