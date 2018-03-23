properties([[
    $class: 'jenkins.model.BuildDiscarderProperty', 
    strategy: [$class: 'LogRotator', numToKeepStr: '5', artifactNumToKeepStr: '5']
]])

@NonCPS
List getJiraIssues() {
    def changeLogSets = currentBuild.changeSets
    def issues = []
    def r = /(CA-[0-9]*)/

    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items

        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]

            echo "MSG from git: " + entry.msg

            if (entry.msg =~ r) {
                def jiraIssue = entry.msg

                echo jiraIssue

                if (!issues.contains(jiraIssue)) {
                    echo "Added " + jiraIssue

                    issues.add(jiraIssue)
                }
            }
        }
    }

    return issues
}
    
node {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm', 'defaultFg': 1, 'defaultBg': 2]) {
        wrap([$class: 'TimestamperBuildWrapper']) {

            catchError {
                stage('checkout scm') {
                    git url: 'git@github.com:rudestan/jenkinspipeline.git'
                    dir ('data/www/test') {
                        checkout scm
                        sh """#!/bin/bash -l
                            git submodule init && git submodule update
                        """
                    }
                }

                stage('JIRA') {
                    echo 'Updating the status of JIRA tickets'
                    def issues = getJiraIssues()
                }                
            }
        }
    }
}
