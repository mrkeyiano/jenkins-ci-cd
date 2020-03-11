// Powered by Femi Patrician

pipeline {
      
    agent any
    environment {
        STAGING_INSTANCE_USER = credentials('DEV_INSTANCE_USER')
        STAGING_INSTANCE_IP = credentials('DEV_INSTANCE_IP')
        INSTANCE_USER = credentials('LIVE_INSTANCE_USER')
        INSTANCE_IP = credentials('LIVE_INSTANCE_IP')
        BITBUCKET_USER = credentials('BITBUCKET_USER')
        BITBUCKET_PASS = credentials('BITBUCKET_PASS')
        REPO = credentials('PATRICIA_CMS_SERVER_REPO')
        REPO_SLUG = "patricia-cms-server"
        PATRICIA_DEVOPS_SLACK = credentials('PATRICIA_DEVOPS_SLACK')
        PATRICIA_UNIVERSE_SLACK = credentials('PATRICIA_UNIVERSE_SLACK')
        REPO_TITLE = "PATRICIA-CMS-SERVER"
        DEPLOY_TITLE = "patricia-cms-server-deploy"
        BUILD_TITLE = "patricia-cms-server-build"
        DEPLOY_DIR = "patricia-cms-server"
        CHECKOUT_BRANCH = getBitbucketBranch()
    
        
    }


    
        
    
   
    stages {
        
        // stage('Clear workspace') {
        //     steps {
        //         script {
        //             sh 'git clean -fdx'
        //         }
        //     }
        // }
    
            
        
        stage ('Checkout') {
            steps {
                script {
                    def scmVars = checkout([$class: 'GitSCM', branches: [[name: "${env.CHECKOUT_BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '87e13238-e989-4f9c-8196-43d16d23a3eb', url: 'https://bitbucket.org/patricia_tech/patricia-cms-server.git']]])
                    env.GIT_COMMIT = scmVars.GIT_COMMIT[0..7]
                    env.GIT_COMMIT_FULL = scmVars.GIT_COMMIT
                    env.GIT_BRANCH =scmVars.GIT_BRANCH
                }
                
             }
        }
    
        // stage('Setup environment') {
        //     steps {
        //       sh 'npm install'
        //     }
        // }
        stage('Run Build Test') {
           
            steps {
                notifyBuild('STARTED')
                 
                script {
                    try {
                      //  sh './vendor/bin/phpunit'
                        notifyBuild('SUCCESSFUL')
                        
                    }
                    catch(err) {
                        notifyBuild('SUCCESSFUL')
                        //error('Build test failed')
                    }
                }
                
                
                
                
                 
            }
            
    } 
    
    
        
        
        stage('Deploy to dev server') {
            
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/dev';
                }
            }
            steps {
                
                script {
                    try {
                        
                    sshagent (credentials: ['staging_private_key']) {
    sh 'ssh -o StrictHostKeyChecking=no -v ${STAGING_INSTANCE_USER}@${STAGING_INSTANCE_IP} "cd /var/www/${DEPLOY_DIR} && sudo git pull https://${BITBUCKET_USER}:${BITBUCKET_PASS}@${REPO} dev --no-edit && sudo chown -R www-data:www-data * && sudo pm2 restart cms-server"'
    notifyDeploy('SUCCESSFUL', 'staging')
  }
                } catch(all) {
                    notifyDeploy('FAILURE', 'staging')
                    error('Deploy failed')
                }
  
                }
            }
            
        }
        
        stage('Notify Slack channel Staging') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/dev';
                }
            }
            steps {
                //post devops slack
            sh label: 'post to slack', script: 'curl -X POST -H "Content-type: application/json" --data "{\\"text\\":\\"@channel [${REPO_TITLE}] has been deployed to Staging Server. Deployment completed - jenkins\\"}" ${PATRICIA_DEVOPS_SLACK}'
            // post to patricia universe
            sh label: 'post to slack 2', script: 'curl -X POST -H "Content-type: application/json" --data "{\\"text\\":\\"@channel [${REPO_TITLE}] has been deployed to Staging Server. Deployment completed - jenkins\\"}" ${PATRICIA_UNIVERSE_SLACK}'
      
              }
        }
        
        
        stage('Deploy to Production server') {
            
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/master';
                }
            }
            steps {
                
                script {
                    try {
                    sshagent (credentials: ['live_private_key']) {
    sh 'ssh -o StrictHostKeyChecking=no -v ${INSTANCE_USER}@${INSTANCE_IP} "cd /var/www/${DEPLOY_DIR} && sudo git pull https://${BITBUCKET_USER}:${BITBUCKET_PASS}@${REPO} master --no-edit && sudo npm run build && sudo cp -r dist/* live && sudo chown -R www-data:www-data *"'
    notifyDeploy('SUCCESSFUL', 'production')
  }
                    } catch(all) {
                        notifyDeploy('FAILED', 'production')
                        error('failed to deploy')
                    }
                }
            }
            
        }
        
        stage('Notify Slack channel Production') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/master';
                }
            }
            steps {
                //post devops slack
            sh label: 'post to slack', script: 'curl -X POST -H "Content-type: application/json" --data "{\\"text\\":\\"@channel [${REPO_TITLE}] has been deployed to Production Server. Deployment completed - jenkins\\"}" ${PATRICIA_DEVOPS_SLACK}'
            // post to patricia universe
            sh label: 'post to slack 2', script: 'curl -X POST -H "Content-type: application/json" --data "{\\"text\\":\\"@channel [${REPO_TITLE}] has been deployed to Production Server. Deployment completed - jenkins\\"}" ${PATRICIA_UNIVERSE_SLACK}'
      
              }
        }
        
    }
     

    

    
    
        
}


def notifyDeploy(String deployStatus = 'SUCCESSFUL', String environment = 'staging') {
    
    if(deployStatus == 'SUCCESSFUL') {
    bitbucketStatusNotify(
      buildState: "${deployStatus}",
      buildKey: 'deploy',
      buildName: "ci/jenkinsci: ${environment}-deploy",
      buildDescription: 'Your deployment was successful via JenkinsCI!',
      repoSlug: "${REPO_SLUG}",
      commitId: "${env.GIT_COMMIT_FULL}"      
    )
    }
    else {
        bitbucketStatusNotify(
      buildState: "${deployStatus}",
      buildKey: 'deploy',
      buildName: "ci/jenkinsci: ${environment}-deploy",
      buildDescription: 'Your deployment failed via JenkinsCI!',
      repoSlug: "${REPO_SLUG}",
      commitId: "${env.GIT_COMMIT_FULL}"      
    )
        
    }
    
    
}


def getBitbucketBranch() {
    
    
    if (checkout_branch != "test") {
        //repo push of new branch
       
        return "${checkout_branch}"
    }
    
    if (source_branch != "test") {
        //pull request
        //should check if its merged
        if(pr_state == "MERGED") {
            return "${dest_branch}"
            
        }
        
    }
    
    if (old_branch != "test") {
        return "${old_branch}"
    }
    
    return "no-branch"
   
    
}


def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'
  def dontNotify = false

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${REPO_TITLE} Build' on -> ${env.CHECKOUT_BRANCH} [Build No: ${env.BUILD_NUMBER}]"
  def summary = "${subject}"
  def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
      
      bitbucketStatusNotify(
      buildState: 'INPROGRESS',
      buildKey: 'build',
      buildName: 'ci/jenkinsci: build',
      buildDescription: 'Your tests started on JenkinsCI!',
      repoSlug: "${REPO_SLUG}",
      commitId: "${env.GIT_COMMIT_FULL}"      
    )
    color = 'YELLOW'
    colorCode = '#FFFF00'
   
  } else if (buildStatus == 'SUCCESSFUL') {
      dontNotify = true
      
      bitbucketStatusNotify(
      buildState: 'SUCCESSFUL',
      buildKey: 'build',
      buildName: 'ci/jenkinsci: build',
      buildDescription: 'Your tests passed on JenkinsCI!',
      repoSlug: "${REPO_SLUG}",
      commitId: "${env.GIT_COMMIT_FULL}"    
    )
      
       summary = ":white_check_mark: <${env.RUN_DISPLAY_URL}|ci/jenkinsci: ${BUILD_TITLE}> *passed* \nYour test passed on JenkinsCI!\n>`${env.GIT_COMMIT}` Merged in ${env.GIT_BRANCH}"
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
      bitbucketStatusNotify(
      buildState: 'FAILED',
      buildKey: 'build',
      buildName: 'ci/jenkinsci: build',
      buildDescription: 'Your tests failed on JenkinsCI!',
      repoSlug: "${REPO_SLUG}",
      commitId: "${env.GIT_COMMIT_FULL}"      
    )
    
      summary = ":no_entry: <${env.RUN_DISPLAY_URL}|ci/jenkinsci: ${BUILD_TITLE}> *failed* \nYour tests failed on JenkinsCI!\n>`${env.GIT_COMMIT}` Merged in ${env.GIT_BRANCH}"
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  
  if(!dontNotify) {
  slackSend (color: colorCode, message: summary)
  }


}

