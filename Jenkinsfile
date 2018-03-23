properties([[
    $class: 'jenkins.model.BuildDiscarderProperty', 
    strategy: [$class: 'LogRotator', numToKeepStr: '5', artifactNumToKeepStr: '5']
]])

node {
    /* Jira config Map */
    def Map jiraConfig = [
        site: 'Jira-CA', 
        regex: /^(CA-[0-9]*).*/, 
        transition: 341, 
        comment: '(/) Build successuful. Build tag: ' + env.BUILD_TAG
    ]

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
                    println 'Updating Jira'

                    def List issues = getJiraIssues(currentBuild.changeSets, jiraConfig.regex)

                    if (issues.size()) {
                        println 'Found ' + issues.size().toString() + ' JIRA tickets to update'

                        updateJiraIssues(issues, jiraConfig.site, jiraConfig.transition, jiraConfig.comment)
                    } else {
                        println 'No JIRA tickets found to update'
                    }
                }                
            }
        }
    }
}

/**
 * Returns list of Jira issues extracted from Git change logs of the current deploy
 *
 * @return List
 */
@NonCPS
List getJiraIssues(List changeSets, String rPattern) {
    def issues = []

    for (int i = 0; i < changeSets.size(); i++) {
        def entries = changeSets[i].items

        for (int j = 0; j < entries.length; j++) {
            def issueMatch = (entries[j].msg =~ rPattern)

            if (issueMatch.matches()) {
                def issueNumber = issueMatch.group(1)

                if (!issues.contains(issueNumber)) {
                    issues.add(issueNumber)
                }
            }
        }
    }

    return issues
}

/**
 * Updates Jira issues, changes the status of correspodning ticket and adds a comment
 *
 * @return boolean
 */
boolean updateJiraIssues(List issues, String jiraSite, int transitionId, String comment) {
    def transitionInput = [transition: [id: transitionId]]

    for(int i = 0; i < issues.size(); i++) {
        def issueKey = issues[i]

        if (transitionId > 0) {
            jiraTransitionIssue idOrKey: issueKey, input: transitionInput, site: jiraSite
        }
        
        jiraAddComment idOrKey: issueKey, comment: comment, site: jiraSite
    }

    return true
}
