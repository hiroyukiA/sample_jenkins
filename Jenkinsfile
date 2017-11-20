#!groovy
node {
  def branch
  def default_branch = env.DEFAULT_BRANCH_GIT ?: 'master'
  def app_repo_url = env.APP_REPO_URL
  stage('Parameters') {
    try{
      timeout(time: 30, unit: 'SECONDS') {
        branch = input message: 'パラメータを入力して下さい',
          parameters: [
            string(defaultValue: default_branch, description: '''master develop''', name: 'branch'),
          ]
      }
    } catch(err) {
      branch = default_branch
    }
  }
  stage('Checkout') {
    checkout([$class: 'GitSCM', branches: [[name: "${branch}"]], userRemoteConfigs: [[url: "${app_repo_url}"]]])
  }
  stage('Build') {
    dir('complete') {
      sh 'docker pull maven:3.5.0-jdk-8-alpine'
      def pwd = pwd()
      withDockerContainer(args: "-v ${pwd}:/usr/src/mymaven -w /usr/src/mymaven", image: 'maven:3.5.0-jdk-8-alpine') {
        sh 'mvn clean'
        sh 'mvn package -DskipTests=true'
      }
    }
  }
  stage('Deploy') {
    sh 'docker pull java:8u111-jdk-alpine'
    sh 'docker ps -a --filter "name=gs-rest-service" | awk \'BEGIN{i=0}{i++;}END{if(i>=2)system("docker stop gs-rest-service")}\''
    sh 'docker run --rm --name gs-rest-service -p 80:80 -v /docker/jenkins/workspace/deploy_app:/usr/src/myapp -w /usr/src/myapp java:8u111-jdk-alpine java -jar complete/target/gs-rest-service-0.1.0.jar --server.port=80 &'
  }
}

