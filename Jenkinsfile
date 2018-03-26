properties([[
    $class: 'jenkins.model.BuildDiscarderProperty', 
    strategy: [$class: 'LogRotator', numToKeepStr: '5', artifactNumToKeepStr: '5']
]])

node {
    /* Jira config Map */
    def Map jiraConfig = [
        site: "Jira-CA", 
        regex: /^(CA-[0-9]*).*/, 
        transition: 341, 
        comment: "(/) Build successufuly deployed.",
        projectId: "10000",
        testingLinkField: "customfield_10300"
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
/*
                stage('JIRA') {
                    println "Updating Jira"

                    def List issues = getJiraIssues(currentBuild.changeSets, jiraConfig.regex)

                    if (issues.size()) {
                        println "Found ${issues.size().toString()} JIRA tickets to update"

                        updateJiraIssues(issues, jiraConfig.site, jiraConfig.transition, jiraConfig.comment)
                    } else {
                        println "No JIRA tickets found"
                    }
                }   */ 

                stage('JIRA Test Link') {
                    def String t = 'CA-5377-some-branch-for-testing'
                    def String jiraIssue = getRegexMatchedStr(t, jiraConfig.regex)

                    if (!jiraIssue) {
                        println "No JIRA tickets found"
                    } else {
                        def String normalizedName = getNormalizedName(t)
                        def String testUrl = "http://go.${normalizedName}.ra.testing.customer-alliance.com"

                        boolean hasTestingUrl = issueHasField(jiraIssue, jiraConfig.testingLinkField, jiraConfig.site)

                        if (!hasTestingUrl) {
                            updateJiraIssueField(testUrl, jiraIssue, jiraConfig.projectId, jiraConfig.testingLinkField, jiraConfig.site)
                        } else {
                            println "Issue already has a test link"
                        }
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
            def issueNumber = getRegexMatchedStr(entries[j].msg, rPattern) 
            if (issueNumber && !issues.contains(issueNumber)) {
                issues.add(issueNumber)
            }
        }
    }

    return issues
}

/**
 * Returns regex matched string 
* 
 * @return String
 */
String getRegexMatchedStr(String sourceString, String rPattern) {
    def strMatch = (sourceString =~ rPattern)

    return strMatch.matches() ? strMatch.group(1) : null
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

/**
 * Updates the value of passed field for Jira issue
 * 
 * @return boolean
 */
boolean updateJiraIssueField(String testUrl, String issueNumber, String projectId, String fieldName, String site) {
    def testIssue = [fields: [
                           project: [id: projectId],
                           "${fieldName}": testUrl]]

    jiraEditIssue idOrKey: issueNumber, issue: testIssue, site: site    
}

/**
 * Checks whether Jira issue already has not null value for certain field
 *
 * @return boolean
 */
boolean issueHasField(String issue, String fieldName, String jiraSite) {
    def jiraIssue = jiraGetIssue idOrKey: issue, site: jiraSite

    return jiraIssue.data.fields."${fieldName}" ?: false
}

/**
 * Returns normalized string
 * 
 * @return String
 */
String getNormalizedName(String branchName) {
    return branchName.toLowerCase().replace(/[^-a-z0-9]/, '-')
}
