def pipeline_output = ""
def send_email(job_name, job_id, job_status, job_output){
    echo("JOB OUTPUT: ${job_output}")
    echo("Job called ${job_name} with Build ID ${job_id} has status ${job_status}")
}


pipeline{
    agent any
    parameters {
        string defaultValue: 'default_name_first', name: 'ARTIFACT_NAME'
        booleanParam description: 'Determines whether the pipeline should fail.', name: 'FAIL_PIPELINE'
        booleanParam 'RUN_TEST'
    }
    stages{
        stage('Download'){
            steps{
                cleanWs()
                echo(message: "DOWNLOAD")
                dir('pipeline_dir'){
                    git(
                        branch: 'pipeline',
                        url: 'https://github.com/KLevon/jenkins-course.git'
                    )
                }
                
                dir('rt'){
                    rtDownload(
                        serverId: '1',
                        spec: '''{
  "files": [
    {
      "pattern": "generic-local/libraries/printer.zip",
      "target": "artifacts/"
    }
  ]
}
'''
                        )
                }
                
                dir('rt'){
                    unzip( zipFile: "artifacts/libraries/printer.zip",
                            dir: """../pipeline_dir/"""
                        )
                }
            }
        }
        stage('Build'){
            steps{
                echo(message: "BUILD")
                dir("pipeline_dir"){
                    bat(script: """
                        Makefile.bat
                    """)
                }
                script{
                    zip(
                        zipFile: "pipeline_dir/zipped",
                        archive: true,
                        dir: "pipeline_dir/"
                        )
                }
            }
        }
        stage('Tests'){
            when{
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps{
                echo("TESTSSSS")
                
                withCredentials(
                    [usernamePassword(credentialsId: 'first_account', passwordVariable: 'psw', usernameVariable: 'usr')]
                    ){
                        echo("The credentials are: ${usr} and ${psw}")
                    }
            }
        }
        stage('Publish'){
            steps{
                echo(message: "PUBLISH")
                rtUpload(
                        serverId: '1',
                        spec: """{
                                  "files": [
                                    {
                                      "pattern": "pipeline_dir/zipped",
                                      "target": "generic-local/release/bojan/${env.BUILD_ID}/${params.ARTIFACT_NAME}"
                                    }
                                  ]
                                }
                                """
                        )
                script{
                    def array = ["printer", "scanner", "main"]
                        
                    for(i = 0; i < 3; i++){
                        dir("pipeline_dir"){
                            pipeline_output += bat(script: """
                                        Tests.bat ${array[i]}
                                        """, returnStdout: true).trim()
                        }
                    }
                    
                    if (params.FAIL_PIPELINE == true){
                        bat(script: """ exit 1
                                    """)
                    }
                }
            }
        }
    }
    post{
        failure{
            send_email(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, pipeline_output)
        }
    }
}
