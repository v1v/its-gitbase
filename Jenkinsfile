// Licensed to Elasticsearch B.V. under one or more contributor
// license agreements. See the NOTICE file distributed with
// this work for additional information regarding copyright
// ownership. Elasticsearch B.V. licenses this file to you under
// the Apache License, Version 2.0 (the "License"); you may
// not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.

@Library('apm@git_base_commit') _

pipeline {
  agent { label 'linux && immutable' }
  environment {
    REPO = 'its-gitbase'
    OWNER = 'v1v'
    BASE_DIR = "src/github.com/${OWNER}/${env.REPO}"
    NOTIFY_TO = credentials('notify-to')
    JOB_GCS_BUCKET = credentials('gcs-bucket')
    JOB_GIT_CREDENTIALS = "f6c7695a-671e-4f4f-a331-acdce44ff9ba"
    PIPELINE_LOG_LEVEL='DEBUG'
  }
  options {
    timeout(time: 1, unit: 'HOURS')
    buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    timestamps()
    ansiColor('xterm')
    disableResume()
    durabilityHint('PERFORMANCE_OPTIMIZED')
    rateLimitBuilds(throttle: [count: 60, durationName: 'hour', userBoost: true])
    quietPeriod(10)
  }
  stages {
    stage('Test Full Custom Checkout') {
      options { skipDefaultCheckout() }
      steps {
        test(basedir: "${BASE_DIR}",
          mergeTarget: "master",
          branch: "${branch}",
          repo: "git@github.com:${OWNER}/${env.REPO}.git",
          credentialsId: "${JOB_GIT_CREDENTIALS}",
          githubNotifyFirstTimeContributor: false,
          reference: "/var/lib/jenkins/${env.REPO}.git"
        )
      }
    }
    stage('Checkout scm') {
      options { skipDefaultCheckout() }
      steps {
        test(basedir: "${BASE_DIR}")
      }
    }
    stage('Checkout scm with extensions') {
      options { skipDefaultCheckout() }
      steps {
        test(basedir: "${BASE_DIR}", reference: "/var/lib/jenkins/${env.REPO}.git")
      }
    }
  }
  post {
    cleanup {
      notifyBuildResult(prComment: true)
    }
  }
}

def getRealCommit(repo, ref){
  def refReplace = ref.replace("/", ".")
  def gitCmd = "git ls-remote -q https://\${GIT_USERNAME}:\${GIT_PASSWORD}@github.com/${OWNER}/${repo}.git ${ref}|sed 's/${refReplace}//'"
  withCredentials([
    usernamePassword(
      credentialsId: "2a9602aa-ab9f-4e52-baf3-b71ca88469c7-UserAndToken",
      passwordVariable: 'GIT_PASSWORD',
      usernameVariable: 'GIT_USERNAME')
  ]) {
    log(level: "INFO", text: "executing: ${gitCmd}")
    return sh(label: 'Get real commit', script: "${gitCmd}", returnStdout: true).trim()
  }
}

def getBranch(){
  def branch = "${env.BRANCH_NAME}"
  if(env.CHANGE_ID){
    branch = "pr/${env.CHANGE_ID}"
  }
  if(env.TAG_NAME?.trim()) {
    branch = "${env.TAG_NAME}"
  }
  return branch
}

def getRef(){
  def ref = "refs/heads/${env.BRANCH_NAME}"
  if(env.CHANGE_ID){
    ref = "refs/pull/${env.CHANGE_ID}/head"
  }
  if(env.TAG_NAME?.trim()) {
    ref = "refs/tags/${env.TAG_NAME}"
  }
  return ref
}

def test(checkoutParams){
  def branch = getBranch()
  def ref = getRef()
  log(level: "INFO", text: "branch: ${branch}")
  log(level: "INFO", text: "ref: ${ref}")

  deleteDir()
  ws(branch){
    String commit = getRealCommit(env.REPO, ref)
    log(level: "INFO", text: "commit: ${commit}")

    gitResetEnvVariables()
    gitCheckout(checkoutParams)
    dir(checkoutParams.basedir) {
      printDebug()
    }
    if (env.TAG_NAME?.trim()) {
      log(level: 'WARN', text: 'GIT_BASE_COMMIT does not support TAGS')
      assertGitBaseCommit(env.GIT_SHA)
    } else {
      assertGitBaseCommit(commit)
    }
  }
}

def assertGitBaseCommit(commit) {
  if(env.GIT_BASE_COMMIT.trim().equalsIgnoreCase(commit.trim())){
    log(level: 'INFO', text: "GIT_BASE_COMMIT value is wrong expected '${commit}' but found '${env.GIT_BASE_COMMIT}'")
  } else {
    error("GIT_BASE_COMMIT value is wrong expected '${commit}' but found '${env.GIT_BASE_COMMIT}'")
  }
}

def gitResetEnvVariables() {
  // In order to run different test scenarios is required to reset the 
  // env variables that are generated on the fly.
  env.GIT_BASE_COMMIT=''
  env.GIT_BUILD_CAUSE=''
  env.GIT_SHA=''
}

def printDebug() {
  sh(label: 'Git env variables', script: 'env | sort | grep -i GIT_')
  sh(label: 'git rev-parse HEAD', script: 'git rev-parse HEAD')
}
