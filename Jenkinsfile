pipeline {
    // 스테이지 별로 다른 거
    agent any//어떤노드쓸껀지 아무거나 하자한거

    triggers {
        pollSCM('*/3 * * * *') //크론 snyt  3분내로 트리거 지정
    }

    environment {  //credentials 에 AWS 환경변수 지정
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')  //새로운 iam의 key로 접근가능하게 
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'eu-west-2'
      HOME = '.' // Avoid npm root owned
    }

    stages {  
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            
            steps {
                echo 'Clonning Repository'

                git url: 'https://github.com/soung9508/test_pipe.git',
                    branch: 'master',
                    credentialsId: 'Jenkins_Git_token'//git에서 연동시킨 id
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }

                always {
                  echo "i tried..." 
                }

                cleanup {
                  echo "after all other post condition"  //해당 스텝이 진행되고나서 post사후처리 됬다는 로그처리
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://testsoom
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository' 

               //   mail  to: 'frontalnh@gmail.com',
               //        subject: "Deploy Frontend Success",
               //         body: "Successfully deployed frontend!"  //성공 실패 배포 이메일  -> access 도 지정해줘야함

              }

              failure {
                  echo 'I failed :('

                //  mail  to: 'frontalnh@gmail.com',
                 //       subject: "Failed Pipelinee",
                 //       body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {  //node서버로 jenkins는 node가 없어서 docker 사용
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }//lint 해주는곳 
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh '''
                docker build . -t server --build-arg env=${PROD}
                ''' //env=${PROD}
            }
          }

          post {
            failure {
              error 'This pipeline stops here...'  //BUILD 잘안됬을때 
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh '''
                docker run -p 80:80 -d server  
                '''
            }//원래 있던 이미지 지우고 새서버 배포    docker rm -f $(docker ps -aq)
          }

          post {
            success {
                echo 'successfully'
          //    mail  to: 'frontalnh@gmail.com',
          //          subject: "Deploy Success",
          //          body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
