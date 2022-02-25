pipeline {
  agent any
    
  tools {nodejs "node"}
    
  stages {

    stage ('Prepare destination host') {
      steps {
        sh '''
          ssh -T centos@192.168.231.144 << ENDSSH
          cd /home/centos
          sudo rm -rf alpha/
          sudo sleep 2
          mkdir alpha
          sudo chmod +x alpha/
          cd alpha/
          mkdir deploy1 deploy2 deploy3
          sudo dnf install npm -y
          sudo npm install
ENDSSH
      '''
      }
    }

    stage('checkout SCM') {
      steps {
        git 'https://github.com/bhimra/alpha-deployment.git'
        sh 'pwd && sudo chmod +x .'
      }
    }

    stage('SCP application on /locations') {
      steps {
        sh 'scp index.js centos@192.168.231.144:/home/centos/alpha/deploy1'
        sh 'scp index2.js centos@192.168.231.144:/home/centos/alpha/deploy2'
        sh 'scp index3.js centos@192.168.231.144:/home/centos/alpha/deploy3'
        sh 'echo "All deployment folder are READY."'
      }
    }   


    // stage('ArchiveArtifacts'){
    //         steps{
    //             archiveArtifacts artifacts: '**', followSymlinks: false
    //         }
    //     }
    
    stage ('Kill existing node service') {
      steps {
        sh '''
          ssh -t -t  centos@192.168.231.144 'bash -s << 'ENDSSH'
          if [[ Z=$(sudo ps aux | grep -i [n]ode | awk 'NR==1' | gawk {'print $2'}) ]];
          then
              sudo kill -9 $Z
              echo "node service stop successfully."
              sudo pkill node
              exit 0
          else
              echo "node service failed"
          fi
ENDSSH'
       '''
       }
    }
    stage ('Stage - 1: Build & Deploy index.js service') {
      steps {
        sh '''
        set -x
        ssh centos@192.168.231.144 "
                    cd /home/centos/alpha/deploy1
                    node index2.js > /dev/null 2>&1 <&- & "
        sudo sleep 3                    
        X=$(curl -k  -o /dev/null -s -w %{http_code} http://192.168.231.144:3000)
        if [ $X -eq 200 ];
            then
                echo -e 'web site on port 3000 is running'
            else
                echo -e 'web site on port 3000 is down' 
        fi '''
       }
      post {
        unstable {
          echo 'The state is Unstable..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            echo -e 'index.js service stop & port 3000 is down'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1'
ENDSSH 
      '''
        }
        failure {
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            echo -e 'index.js service stop & port 3000 is down'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1'
ENDSSH 
      '''
        }
      }
    }
    stage ('Stage - 2: Build & Deploy index2.js service') {
      steps {
        sh '''
        set -x
        ssh centos@192.168.231.144 "
                    cd /home/centos/alpha/deploy2
                    node index2.js > /dev/null 2>&1 <&- & "
        sudo sleep 3                    
        X=$(curl -k  -o /dev/null -s -w %{http_code} http://192.168.231.144:3002)
        if [ $X -eq 200 ];
            then
                echo -e 'web site on port 3002 is running'
            else
                echo -e 'web site on port 3002 is down' 
        fi '''
       }
      post {
        unstable {
          echo 'The state is Unstable..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            netstat -anp 2> /dev/null | grep :3000 | awk '{ print $7 }' | cut -d'/' -f1 | xargs kill
            echo -e 'index.js service stop & port 3000 is down...............................'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1...........................................'
            echo -e 'index.js service stop & port 3002 is down.........................'
            sudo rm -rf /home/centos/alpha/deploy2/* 
            echo -e 'removed index2.js from deploy2...........................................'
ENDSSH 
      '''
        }
        failure {
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            netstat -anp 2> /dev/null | grep :3000 | awk '{ print $7 }' | cut -d'/' -f1 | xargs kill
            echo -e 'index.js service stop & port 3000 is down...............................'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1...........................................'
            sudo rm -rf /home/centos/alpha/deploy2/* 
            echo -e 'removed index.js from deploy2...........................................'
ENDSSH 
      '''
        }
      }
    }
    
    stage ('Stage - 3: Build & Deploy index3.js service') {
      steps {
        sh '''
        set -x
        ssh centos@192.168.231.144 "
                    cd /home/centos/alpha/deploy3
                    node index3.js > /dev/null 2>&1 <&- & "
        sudo sleep 5                    
        X=$(curl -k  -o /dev/null -s -w %{http_code} http://192.168.231.144:3003)
        if [ $X -eq 200 ];
            then
                echo -e 'web site on port 3003 is running'
            else
                echo -e 'web site on port 3003 is down' 
        fi '''
       }
      post {
        unstable {
          echo 'The state is Unstable..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            netstat -anp 2> /dev/null | grep :3000 | awk '{ print $7 }' | cut -d'/' -f1 | xargs kill
            echo -e 'index.js service stop & port 3000 is down...............................'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1...........................................'
            netstat -anp 2> /dev/null | grep :3002 | awk '{ print $7 }' | cut -d'/' -f1 | xargs kill
            echo -e 'index.js service stop & port 3002 is down...............................'
            sudo rm -rf /home/centos/alpha/deploy2/* 
            echo -e 'removed index.js from deploy2...........................................'
ENDSSH 
      '''
        }
        failure {
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            netstat -anp 2> /dev/null | grep :3000 | awk '{ print $7 }' | cut -d'/' -f1 | xargs kill
            echo -e 'index.js service stop & port 3000 is down...............................'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1...........................................'
            netstat -anp 2> /dev/null | grep :3002 | awk '{ print $7 }' | cut -d'/' -f1 | xargs kill
            echo -e 'index.js service stop & port 3002 is down...............................'
            sudo rm -rf /home/centos/alpha/deploy2/* 
            echo -e 'removed index.js from deploy2...........................................'
ENDSSH 
      '''
         }
       }
     }
  }
}
