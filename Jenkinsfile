properties([
  parameters([
    choice(
      choices: [
        'Development+Shipping',
        'Development+Test+Shipping',
        'Development',
        'Shipping',
        'Test',
        'DebugGame'
      ],
      description: '''Development = This configuration is equivalent to Release. Unreal Editor uses the Development configuration by default. Compiling your project using the Development configuration enables you to see code changes made to your project reflected in the editor.\n\nShipping =  This is the configuration for optimal performance and shipping your game. This configuration strips out console commands, stats, and profiling tools.\n\nTest =  This configuration is the Shipping configuration, but with some console commands, stats, and profiling tools enabled.\n\nDebugGame = This configuration builds the engine as optimized, but leaves the game code debuggable. This configuration is ideal for debugging only game modules.''',
      name: 'BUILD_CONFIGURATION'
    ),
    string(
      defaultValue: 'Providence',
      description: 'The name of the uproject file.',
      name: 'PROJECT_NAME'
    ),
    booleanParam(
      description: 'Removes the derived data cached folders and some cached data in the Saved folders. Use this if there are visual discrepancies between a local standalone build and the one in Jenkins. For example, some shaders stop working in a Jenkins build.',
      name: 'REMOVE_CACHED_DATA'
    ),
    string(
      defaultValue: 'D:\\Jenkins\\Archive',
      description: 'The path where builds will be archive in the Jenkins agent.',
      name: 'PATH_TO_ARCHIVE'
    ),
    string(
      defaultValue: 'D:\\Jenkins\\temp',
      description: 'The path where temp builds will be generated. From this folder they will be compressed and moved to the Archive folder',
      name: 'PATH_TO_TEMP'
    ),
    booleanParam(
      defaultValue: true,
      description: 'If set to be fast Binaries and Intermediate folders won\'t be deleted. By default this is FALSE as we want to have a clean build.',
      name: 'FAST_BUILD'
    ),
    [
      $class: 'CascadeChoiceParameter', 
      choiceType: 'PT_CHECKBOX',
      description: 'Select the maps to be cooked.',
      filterLength: 1,
      filterable: false,
      name: 'MAPS_TO_COOK',
      script: [
        $class: 'GroovyScript',
        fallbackScript: [
          classpath: [], 
          sandbox: false, 
          script: 'return ["ERROR"]'
        ],
        script: [
          classpath: [], 
          sandbox: false, 
          script: """
            List maps = [];new File("C:\\Users\\Merih Dereli\\AppData\\Local\\Jenkins\\.jenkins\\workspace\\multibranch_master\\param-test.txt").eachLine { line -> maps.add(line)};return maps;
          """.stripIndent()
        ]
      ]
    ],
    booleanParam (
      name: 'REMOVE_DEV_FOLDERS_KEY',
      defaultValue: false,
      description: 'Select to remove +DirectoriesToNeverCook key from DefaultGame.ini file.'
    )
  ])
])

pipeline {
  agent any
  environment {
    CREDENTIALS_ID = 'google-storage-test'
    BUCKET = 'testo-jenkins-build-bucket'
    PATTERN = '*.class'
    JENKINS_API_USER = credentials('jenkins_api_user')
    JENKINS_API_TOKEN = credentials('jenkins_api_token')
  }
  stages {
    stage('Compile') {
      steps {
        echo 'Compiling...'
        sh 'javac HelloWorld.java'
        sh 'java HelloWorld'
      }
    }
    stage('Store to GCS') {
      steps {
        // step([$class: 'ClassicUploadStep', credentialsId: env.CREDENTIALS_ID, bucket: "gs://${env.BUCKET}", pattern: env.PATTERN])
        echo "ini-customizer -ini Game/Providence/Config/DefaultGame.ini -removedevfolderskey=${params.REMOVE_DEV_FOLDERS_KEY} -maps_to_cook ${params.MAPS_TO_COOK}"
      }
    }
    stage('Send notification to Slack') {
      steps {
        // slackSend color: "#439FE0", message: "Build info: ${env.JOB_NAME} ${env.BUILD_NUMBER} <${env.BUILD_URL}|Open>"
        echo 'send notification to slack commented for this step'
      }
    }
  }
  post {
    always {
      echo 'pipeline always echoes this message'
      // sh 'curl -s -o build_log_$BUILD_NUMBER.txt --user $JENKINS_API_USER:$JENKINS_API_TOKEN \"$BUILD_URL/consoleText\"'
    }
    success {
      echo 'pipeline finished successfully'
    }
    failure {
      // slackSend color: "#c91a1a", message: "ERROR: Pipeline failed."
      // slackUploadFile filePath: "build_log_*.txt", initialComment: "See attached build logs for details."
      echo 'job failed'
    }
    cleanup {
      // sh 'rm build_log_$BUILD_NUMBER.txt'
      echo "cleanup"
    }
  }
}